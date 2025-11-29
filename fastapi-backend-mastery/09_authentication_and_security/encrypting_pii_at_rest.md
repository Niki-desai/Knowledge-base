# Encrypting PII at Rest

Protecting Personally Identifiable Information (PII) at rest is critical for compliance and security.

## Encryption at Rest

```python
from cryptography.fernet import Fernet
from sqlalchemy import Column, String, LargeBinary

# Generate key (store securely!)
key = Fernet.generate_key()
cipher = Fernet(key)

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email_encrypted = Column(LargeBinary)  # Encrypted email
    phone_encrypted = Column(LargeBinary)  # Encrypted phone
    
    @property
    def email(self) -> str:
        """Decrypt email when accessed."""
        return cipher.decrypt(self.email_encrypted).decode()
    
    @email.setter
    def email(self, value: str):
        """Encrypt email when set."""
        self.email_encrypted = cipher.encrypt(value.encode())
```

## Field-Level Encryption

```python
from sqlalchemy.types import TypeDecorator, LargeBinary

class EncryptedString(TypeDecorator):
    """SQLAlchemy type for encrypted strings."""
    impl = LargeBinary
    cache_ok = True
    
    def process_bind_param(self, value, dialect):
        if value is not None:
            return cipher.encrypt(value.encode())
        return value
    
    def process_result_value(self, value, dialect):
        if value is not None:
            return cipher.decrypt(value).decode()
        return value

class User(Base):
    email = Column(EncryptedString)  # Automatically encrypted/decrypted
```

## Key Management

```python
import os
from cryptography.fernet import Fernet

# Load key from environment (use secrets manager in production)
ENCRYPTION_KEY = os.getenv("ENCRYPTION_KEY")
if not ENCRYPTION_KEY:
    raise ValueError("ENCRYPTION_KEY not set")

cipher = Fernet(ENCRYPTION_KEY.encode())
```

Encrypt PII at rest to protect sensitive data!

