# Mocking External APIs and AI

Mocking external APIs and AI services makes tests fast and reliable.

## Mocking HTTP Clients

```python
from unittest.mock import AsyncMock, patch
import httpx

@pytest.mark.asyncio
async def test_external_api_call():
    """Test with mocked external API."""
    with patch('httpx.AsyncClient.get') as mock_get:
        # Setup mock response
        mock_response = AsyncMock()
        mock_response.json.return_value = {"status": "ok"}
        mock_response.status_code = 200
        mock_get.return_value = mock_response
        
        # Test
        async with httpx.AsyncClient() as client:
            response = await client.get("https://api.example.com/data")
        
        # Assert
        assert response.status_code == 200
        mock_get.assert_called_once()
```

## Mocking OpenAI

```python
from unittest.mock import AsyncMock

@pytest.fixture
def mock_openai_client():
    """Mock OpenAI client."""
    client = AsyncMock()
    
    # Mock embeddings response
    client.embeddings.create = AsyncMock(return_value=type('obj', (object,), {
        'data': [type('obj', (object,), {'embedding': [0.1] * 1536})()]
    })())
    
    return client

@pytest.mark.asyncio
async def test_generate_embedding(mock_openai_client):
    """Test embedding generation with mock."""
    embedding = await generate_embedding("test", client=mock_openai_client)
    assert len(embedding) == 1536
```

## Using Responses Library

```python
import responses

@responses.activate
def test_external_api():
    """Test with responses library."""
    responses.add(
        responses.GET,
        "https://api.example.com/data",
        json={"status": "ok"},
        status=200
    )
    
    response = requests.get("https://api.example.com/data")
    assert response.json()["status"] == "ok"
```

Mock external services for fast, reliable tests!

