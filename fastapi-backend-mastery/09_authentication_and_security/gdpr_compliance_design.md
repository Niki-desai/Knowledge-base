# GDPR Compliance Design

Designing for GDPR compliance ensures data privacy and regulatory compliance.

## Right to Access (Article 15)

```python
@router.get("/users/{user_id}/data")
async def get_user_data(user_id: int, current_user: User = Depends(get_current_user)):
    """Export all user data (GDPR Right to Access)."""
    if current_user.id != user_id:
        raise HTTPException(403, "Not authorized")
    
    # Collect all user data
    user_data = {
        "profile": await get_user_profile(user_id),
        "orders": await get_user_orders(user_id),
        "activity_log": await get_user_activity(user_id)
    }
    
    return user_data
```

## Right to Erasure (Article 17)

```python
@router.delete("/users/{user_id}/data")
async def delete_user_data(user_id: int, current_user: User = Depends(get_current_user)):
    """Delete all user data (GDPR Right to Erasure)."""
    if current_user.id != user_id:
        raise HTTPException(403, "Not authorized")
    
    # Soft delete (mark as deleted, anonymize)
    await anonymize_user_data(user_id)
    
    return {"status": "deleted"}
```

## Data Minimization

```python
# Only collect necessary data
class UserCreate(BaseModel):
    email: EmailStr  # Required
    name: str  # Required
    # Don't collect unnecessary fields
```

## Audit Logging

```python
class DataAccessLog(Base):
    __tablename__ = "data_access_logs"
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer)
    access_type = Column(String(50))  # read, update, delete
    accessed_by = Column(Integer)  # Who accessed
    timestamp = Column(DateTime, default=datetime.utcnow)
```

GDPR-compliant design protects user privacy and ensures compliance!

