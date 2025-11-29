# Async SQLAlchemy Deep Dive

SQLAlchemy's async support enables non-blocking database operations in FastAPI. This guide covers async SQLAlchemy patterns and best practices.

## Async SQLAlchemy Basics

### Engine Setup

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# Async engine with asyncpg driver
engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    echo=True,
    future=True
)

# Async session maker
async_session_maker = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)
```

### Basic Async Operations

```python
async def get_user(user_id: int, session: AsyncSession):
    # Async query
    result = await session.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()

async def create_user(user_data: dict, session: AsyncSession):
    user = User(**user_data)
    session.add(user)
    await session.flush()  # Get ID without committing
    await session.refresh(user)  # Load relationships
    await session.commit()
    return user
```

## Key Async Patterns

### 1. **Using select() Instead of Query**

```python
# ✅ Async: Use select()
from sqlalchemy import select

result = await session.execute(
    select(User).where(User.email == "user@example.com")
)
user = result.scalar_one_or_none()

# ❌ Sync: Don't use query()
# user = session.query(User).filter(...).first()  # Doesn't work in async
```

### 2. **Async Relationships**

```python
class User(Base):
    __tablename__ = "users"
    id: int
    orders: List["Order"] = relationship("Order", lazy="selectin")

# Load relationships
result = await session.execute(
    select(User).options(selectinload(User.orders))
)
user = result.scalar_one()
# user.orders is already loaded (no additional query)
```

### 3. **Bulk Operations**

```python
# Bulk insert
users_data = [{"email": f"user{i}@example.com"} for i in range(100)]
await session.execute(
    insert(User).values(users_data)
)
await session.commit()

# Bulk update
await session.execute(
    update(User)
    .where(User.is_active == False)
    .values(is_active=True)
)
await session.commit()
```

## Advanced Async Patterns

### Eager Loading

```python
from sqlalchemy.orm import selectinload, joinedload

# Selectin loading (separate query)
result = await session.execute(
    select(User)
    .options(selectinload(User.orders))
)
user = result.scalar_one()

# Joined loading (single query with JOIN)
result = await session.execute(
    select(User)
    .options(joinedload(User.orders))
)
user = result.scalar_one()
```

### Streaming Large Results

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

async def stream_users(session: AsyncSession):
    """Stream users in batches"""
    async for batch in await session.stream(
        select(User).order_by(User.id)
    ):
        for user in batch.scalars():
            yield user
```

## Common Pitfalls

### 1. **Blocking Operations**

```python
# ❌ Bad: Blocking operation
result = session.query(User).all()  # Blocking!

# ✅ Good: Async operation
result = await session.execute(select(User))
users = result.scalars().all()
```

### 2. **Lazy Loading Issues**

```python
# ❌ Bad: Lazy load after session closed
user = await session.get(User, 1)
await session.close()
print(user.orders)  # Error: session closed

# ✅ Good: Eager load
user = await session.execute(
    select(User).options(selectinload(User.orders))
)
user = result.scalar_one()
await session.close()
print(user.orders)  # OK: already loaded
```

### 3. **Transaction Management**

```python
# ❌ Bad: Missing commit
user = User(email="test@example.com")
session.add(user)
# Missing: await session.commit()

# ✅ Good: Proper transaction
try:
    session.add(user)
    await session.commit()
except Exception:
    await session.rollback()
    raise
```

## Best Practices

1. Always use `select()` instead of `query()`
2. Use `scalar_one_or_none()` instead of `first()`
3. Eager load relationships to avoid lazy loading issues
4. Use bulk operations for multiple rows
5. Properly manage transactions (commit/rollback)
6. Close sessions explicitly or use context managers

## Summary

Async SQLAlchemy provides:
- Non-blocking database operations
- Better concurrency handling
- Integration with FastAPI's async architecture

Key differences from sync SQLAlchemy:
- Use `select()` instead of `query()`
- Use `await` for all operations
- Eager load relationships
- Proper transaction management

