# 01. FastAPI Core Concepts: The Pillars of Power

## 1. Pydantic: Data Validation on Steroids

FastAPI doesn't just "use" Pydantic; it is fused with it. Pydantic is the reason you don't write boilerplate validation code.

### The Magic of Parsing
It doesn't just check types; it **coerces** them.
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = None

# Input: {"name": "Apple", "price": "5.5", "is_offer": "True"}
# Output: name="Apple", price=5.5, is_offer=True
```
It turned the string "5.5" into a float and "True" into a boolean automatically.

### Field Validation
You can add custom logic easily.
```python
from pydantic import validator

class User(BaseModel):
    age: int

    @validator('age')
    def check_age(cls, v):
        if v < 18:
            raise ValueError('Must be 18+')
        return v
```

---

## 2. Dependency Injection: The Secret Weapon

Most frameworks make DI hard. FastAPI makes it trivial.

### What is it?
"I need X to do my job. Don't tell me how to build X, just give it to me."

### The `Depends` System
It's a hierarchical graph resolution system.
1.  Route A needs `User`.
2.  `User` needs `Token`.
3.  `Token` needs `Header`.

FastAPI resolves this graph, executes dependencies in order, and passes the results down.

```python
def get_token(header: str = Header(...)):
    return header

def get_user(token: str = Depends(get_token)):
    return find_user(token)

@app.get("/me")
def read_me(user: User = Depends(get_user)):
    return user
```
**Why is this amazing?**
- **Reusability**: `get_user` can be used in 50 different routes.
- **Testing**: You can override `get_user` with a mock during tests.

---

## 3. Async/Await: Concurrency for Humans

FastAPI is built on **Starlette** and **AnyIO**, making it natively asynchronous.

### `def` vs `async def`
- **`async def`**: Use this if your code uses `await` (e.g., DB calls, API requests). It runs on the main event loop. **FAST**.
- **`def`**: Use this if your code is blocking (e.g., heavy math, standard `time.sleep`). FastAPI runs this in a separate thread pool (threadpool) to prevent blocking the loop. **SAFE**.

**The Golden Rule**: If you are using a library that blocks (like `requests` or standard `sqlite3`), use `def`. If you use `httpx` or `motor`, use `async def`.

---

## 4. Path & Query Parameters

### Path Parameters (`/users/{id}`)
Mandatory parts of the route.
```python
@app.get("/items/{item_id}")
def read_item(item_id: int): # Type conversion happens here!
    return {"item_id": item_id}
```

### Query Parameters (`/users?skip=0&limit=10`)
Optional configuration.
```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

### Validation
You can enforce strict rules on parameters using `Query` and `Path`.
```python
from fastapi import Query

@app.get("/items/")
def read_items(q: str = Query(None, min_length=3, max_length=50, regex="^fixedquery$")):
    ...
```

---

## 5. Middleware: The Gatekeepers

Middleware runs **before** the request hits your route and **after** the response is generated.

**Use cases:**
- **CORS**: Allowing your frontend to talk to your backend.
- **Process Time Header**: Measuring how long a request took.
- **GZip**: Compressing responses.

```python
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request) # Pass to route
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```
