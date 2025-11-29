# Testing Async Repositories

Testing async repositories requires proper async test setup and mocking.

## Basic Async Repository Test

```python
import pytest
from unittest.mock import AsyncMock
from app.repositories.user_repository import UserRepository

@pytest.mark.asyncio
async def test_get_user_by_id(mock_db_session):
    """Test getting user by ID."""
    repo = UserRepository(mock_db_session)
    
    # Mock database response
    expected_user = User(id=1, email="test@example.com")
    mock_db_session.get = AsyncMock(return_value=expected_user)
    
    # Test
    user = await repo.get_by_id(1)
    
    # Assert
    assert user == expected_user
    mock_db_session.get.assert_called_once_with(User, 1)
```

## Testing with Test Database

```python
@pytest.mark.asyncio
async def test_create_user_integration(db_session):
    """Integration test with real database."""
    repo = UserRepository(db_session)
    
    # Create user
    user = await repo.create(
        email="test@example.com",
        full_name="Test User"
    )
    
    # Verify
    assert user.id is not None
    assert user.email == "test@example.com"
    
    # Verify in database
    found = await repo.get_by_id(user.id)
    assert found is not None
```

## Testing Query Methods

```python
@pytest.mark.asyncio
async def test_find_users_by_email(db_session):
    """Test finding users by email pattern."""
    repo = UserRepository(db_session)
    
    # Create test users
    await repo.create(email="john@example.com", full_name="John")
    await repo.create(email="jane@example.com", full_name="Jane")
    await db_session.commit()
    
    # Test
    users = await repo.find_by_email_pattern("john")
    
    # Assert
    assert len(users) == 1
    assert users[0].email == "john@example.com"
```

Test async repositories with proper async test patterns!

