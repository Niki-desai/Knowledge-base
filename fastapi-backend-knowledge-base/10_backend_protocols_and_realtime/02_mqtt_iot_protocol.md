# 02. MQTT Deep Dive: Reliable Messaging for Unreliable Networks

## 1. The Philosophy of "Lightweight"

HTTP is fat. It sends headers, cookies, and metadata with every request.
MQTT is anorexic. The smallest packet is just **2 bytes**.

This matters when your "server" is a lightbulb with 32KB of RAM running on a coin battery.

---

## 2. Quality of Service (QoS): The Contract

QoS is the most misunderstood part of MQTT. It's not about speed; it's about **certainty**.

### QoS 0: "At Most Once" (Fire & Forget)
- **Mechanism**: Sender transmits packet. Done.
- **Overhead**: Lowest.
- **Risk**: If the network blips, the message is gone forever.
- **Use Case**: Sensor readings (Temperature updates every second. Missing one is fine).

### QoS 1: "At Least Once" (Acknowledged)
- **Mechanism**: Sender transmits. Stores message locally. Waits for `PUBACK`. If no Ack, resends.
- **Overhead**: Medium.
- **Risk**: **Duplicate Messages**. If the Ack gets lost, the sender resends. Your consumer must be **Idempotent**.
- **Use Case**: "Turn on the lights". You definitely want the lights on, and turning them on twice is harmless.

### QoS 2: "Exactly Once" (The Handshake)
- **Mechanism**: 4-step handshake (`PUBLISH` -> `PUBREC` -> `PUBREL` -> `PUBCOMP`).
- **Overhead**: Highest. Slowest.
- **Risk**: None.
- **Use Case**: Financial transactions, Critical Alarms.

---

## 3. Advanced Features

### Retained Messages: "What did I miss?"
A new subscriber connects. It wants to know the *current* state of the system.
- Without Retain: It waits for the next update (could be hours).
- **With Retain**: The broker immediately sends the last known good value.
*Example*: A dashboard app connects and immediately shows "Living Room Light: ON".

### Last Will and Testament (LWT)
The "Dead Man's Switch".
When a client connects, it tells the broker: *"If I die unexpectedly (connection drop, power loss), please publish 'OFFLINE' to topic `status/my-device`."*
This allows the system to detect device failures instantly, rather than waiting for a timeout.

### Persistent Sessions (`clean_session=False`)
If a client disconnects and reconnects later:
- **Clean Session**: Start fresh. Missed messages are lost.
- **Persistent Session**: The broker queues messages *while the client is offline*. When it reconnects, it gets a flood of missed data.

---

## 4. FastAPI + MQTT: The Architecture

FastAPI is a Request/Response server. MQTT is an Event stream. Mixing them requires a **Bridge**.

### Pattern 1: The Sidecar Worker
Don't run MQTT inside the FastAPI process (Uvicorn). Run it as a separate process.

1.  **FastAPI**: Handles HTTP. Writes commands to Redis/Database.
2.  **MQTT Worker**: Reads from Redis/Database -> Publishes to MQTT. Listens to MQTT -> Writes to Database.

### Pattern 2: The Loop (Small Scale)
Running the loop in a background thread within FastAPI (as shown in the basic example) is risky.
- If FastAPI restarts, you lose the MQTT connection.
- If you run 4 Uvicorn workers, you have 4 MQTT connections (and 4 duplicate messages if not careful!).

**Recommendation**: Use **Pattern 1** for production.

---

## 5. Security: TLS and Auth

**Never** run MQTT on port 1883 (plaintext) over the public internet.
1.  **TLS/SSL (Port 8883)**: Encrypts the payload.
2.  **Authentication**: Username/Password is standard.
3.  **ACL (Access Control Lists)**:
    - User `sensor_1` can PUBLISH to `sensors/temp`.
    - User `dashboard` can SUBSCRIBE to `sensors/#`.
    - User `hacker` cannot do anything.

Mosquitto configuration (`mosquitto.conf`) handles this.

```conf
listener 8883
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate true
```
