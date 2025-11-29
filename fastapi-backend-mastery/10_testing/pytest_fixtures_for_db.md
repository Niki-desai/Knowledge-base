# Pytest Fixtures for Database Testing

Proper fixtures make database testing clean and efficient.

## Database Session Fixture

```python
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

@pytest.fixture(scope="session")
async def test_engine():
    """Create test database engine."""
    engine = create_async_engine("postgresql+asyncpg://test:test@localhost/test_db")
    
    # Create tables
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    yield engine
    
    # Cleanup
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    await engine.dispose()

@pytest.fixture
async def db_session(test_engine):
    """Create database session for each test."""
    async_session_maker = async_sessionmaker(test_engine, expire_on_commit=False)
    
    async with async_session_maker() as session:
        yield session
        await session.rollback()  # Rollback uncommitted changes
```

## Transaction Rollback Fixture

```python
@pytest.fixture
async def db_session_transaction(test_engine):
    """Session with transaction that rolls back."""
    connection = await test_engine.connect()
    transaction = await connection.begin()
    
    session = AsyncSession(bind=connection)
    
    yield session
    
    await session.close()
    await transaction.rollback()
    await connection.close()
```

## Clean Database Fixture

```python
@pytest.fixture
async def clean_db(db_session):
    """Clean database before each test."""
    # Delete all data
    for table in reversed(Base.metadata.sorted_tables):
        await db_session.execute(table.delete())
    await db_session.commit()
    
    yield db_session
```

Use fixtures for clean, isolated database tests!

