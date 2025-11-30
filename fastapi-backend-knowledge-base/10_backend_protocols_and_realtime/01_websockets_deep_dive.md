# 01. WebSockets Deep Dive: Scaling, Security, and Resilience

## 1. Beyond the Basics: The Reality of Real-Time

A simple `WebSocket` endpoint works for a demo. But in production, you face three killers:
1.  **State**: WebSockets are stateful. If your server restarts, connections die.
2.  **Scaling**: You can't just spin up 10 servers. User A on Server 1 cannot talk to User B on Server 2.
3.  **Security**: Standard HTTP headers (like `Authorization`) don't work the same way in the JS WebSocket API.

This guide moves beyond "Hello World" to "Production Ready".

---

## 2. Scaling: The Redis Pub/Sub Pattern

To make multiple FastAPI workers talk to each other, we need a **Backplane**. Redis is the industry standard for this.

### The Architecture
- **User A** connects to **Worker 1**.
- **User B** connects to **Worker 2**.
- **Worker 1** publishes a message to Redis Channel `chat_room_1`.
- **Worker 2** is subscribed to `chat_room_1`. It receives the message and pushes it to User B.

### Implementation with `broadcaster`

```python
# pip install broadcaster[redis]
from broadcaster import Broadcast
from fastapi import FastAPI, WebSocket

broadcast = Broadcast("redis://localhost:6379")
app = FastAPI(on_startup=[broadcast.connect], on_shutdown=[broadcast.disconnect])

@app.websocket("/ws/{room_id}")
async def chatroom(websocket: WebSocket, room_id: str):
    await websocket.accept()
    
    # 1. Subscribe to the Redis channel for this room
    async with broadcast.subscribe(channel=room_id) as subscriber:
        try:
            while True:
                # 2. Listen for incoming messages from Client OR Redis
                # This is tricky: we need to listen to TWO things at once.
                # We use asyncio.create_task or a select-like pattern.
                
                # Simplified flow:
                data = await websocket.receive_text()
                
                # 3. Publish to Redis (so all workers see it)
                await broadcast.publish(channel=room_id, message=data)
                
                # Note: The subscriber loop handles sending Redis msgs -> Client
        except Exception:
            pass
```

*Note: Real implementation requires careful `asyncio` task management to listen to both the websocket and the redis subscriber simultaneously.*

---

## 3. Authentication: The Header Problem

The standard JavaScript `new WebSocket(url)` API **does not allow custom headers**. You cannot send `Authorization: Bearer xyz`.

### Strategy A: Query Parameters (Easiest)
`ws://api.com/ws?token=xyz`
- **Pros**: Works everywhere.
- **Cons**: Token is visible in server logs (URL logging). **Risk**.

### Strategy B: The "Ticket" System (Best Practice)
1.  Client calls `POST /auth/ticket` (Standard HTTP with Auth Header).
2.  Server validates and returns a short-lived `ticket_id` (e.g., valid for 10 seconds).
3.  Client connects `ws://api.com/ws?ticket=ticket_id`.
4.  Server validates ticket and upgrades connection.

### Strategy C: Protocol Level Auth
1.  Connect unauthenticated.
2.  First message MUST be `{"type": "auth", "token": "xyz"}`.
3.  Server validates. If fail, close connection.

```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    
    # Enforce Auth Handshake
    auth_data = await websocket.receive_json()
    if auth_data.get("type") != "auth":
        await websocket.close(code=1008) # Policy Violation
        return
        
    token = auth_data.get("token")
    user = validate_token(token)
    if not user:
        await websocket.close(code=1008)
        return
        
    # Proceed to main loop...
```

---

## 4. Resilience: Heartbeats and Reconnection

Connections *will* drop. Wi-Fi flickers, mobile networks switch towers.

### The Ping/Pong
WebSockets have a built-in Ping/Pong frame, but sometimes "zombie" connections stay open (half-open TCP).
**Application Level Heartbeat**:
- Server sends `{"type": "ping"}` every 30s.
- Client must reply `{"type": "pong"}`.
- If no pong in 10s, Server kills connection.

### Exponential Backoff (Client Side)
When disconnected, **do not** reconnect immediately. You will DDoS your own server.
- Attempt 1: Wait 1s
- Attempt 2: Wait 2s
- Attempt 3: Wait 4s
- Max: 30s

---

## 5. Inductive Example: The "Typing..." Indicator

A classic real-time feature that requires efficiency. You don't want to broadcast "User is typing" on *every keystroke*.

**Logic:**
1.  User presses key.
2.  Client sends `typing_start` event.
3.  Client sets a "debounce" timer (e.g., 1000ms).
4.  If user types again, reset timer.
5.  If timer expires, send `typing_stop`.

**Server Optimization:**
Don't save "typing" state to the DB. It's ephemeral. Just broadcast it to the room. If a packet is lost, nobody cares. This is a **QoS 0** (Fire and Forget) scenario.
