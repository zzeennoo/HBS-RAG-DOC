This document outlines the API changes introduced in the "Refactor API with logging, streaming, and management features" pull request. It includes details on new endpoints, modified endpoints, usage examples, and clearly marked breaking changes.

---

## API Documentation: Refactor with Logging, Streaming, and Management Features

### Overview

This refactor introduces several new features to the API, including:
*   **Logging Middleware**: A new middleware for logging incoming requests.
*   **Health Check**: A dedicated endpoint to monitor API health.
*   **Enhanced Upload**: Support for uploading multiple documents simultaneously.
*   **Streaming Chat**: A new endpoint for real-time, streaming chat responses.
*   **System Management**: An endpoint to reset the RAG system.
*   **General Streaming**: A generic endpoint for server-sent event (SSE) streaming.

### Breaking Changes

**Please review these changes carefully as they may require updates to existing client implementations.**

*   **`/upload` Endpoint Signature Change**:
    *   **Old**: `POST /upload` expected a single `UploadFile`.
    *   **New**: `POST /upload` now expects a `List[UploadFile]`.
    *   **Impact**: Clients sending a single file without wrapping it in a list will receive an error (e.g., `422 Unprocessable Entity`).
    *   **Action Required**: Update clients to send a list of files, even if it's a list containing a single file.

*   **`/chat` Endpoint Behavior Change**:
    *   **Old**: `POST /chat` returned a direct JSON response.
    *   **New**: `POST /chat` now returns a `StreamingResponse` for real-time, token-by-token output.
    *   **Impact**: Clients expecting a single JSON object will need to adapt to consume a stream of Server-Sent Events (SSE).
    *   **Action Required**: Update clients to handle SSE. If a non-streaming chat response is still required, use the new `/chat_response` endpoint.

---

### New Endpoints

#### 1. Health Check

Checks the operational status of the API.

*   **Endpoint**: `GET /health`
*   **Description**: Provides a simple health check to determine if the API is running and responsive.
*   **Parameters**: None
*   **Returns**:
    *   `200 OK` with a JSON object indicating the status.

**Example Response:**

```json
{
  "status": "healthy"
}
```

**Usage Example (Python - `httpx`):**

```python
import httpx

async def check_health():
    async with httpx.AsyncClient() as client:
        response = await client.get("http://localhost:8000/health")
        response.raise_for_status()
        print(response.json())

# Output: {'status': 'healthy'}
```

#### 2. Streaming Chat

Initiates a chat session with the RAG system, providing responses as a stream of tokens.

*   **Endpoint**: `POST /chat`
*   **Description**: Sends a user message to the RAG system and receives a real-time stream of the generated response. This endpoint is designed for interactive, token-by-token display.
*   **Parameters**:
    *   `request`: `ChatRequest` (JSON body)
        *   `message`: `str` - The user's message.
        *   `session_id`: `Optional[str]` - An optional session ID to maintain conversation context.
*   **Returns**:
    *   `200 OK` with a `StreamingResponse` (Server-Sent Events - SSE). Each event will contain a chunk of the response.

**`ChatRequest` Model:**

```python
from pydantic import BaseModel
from typing import Optional

class ChatRequest(BaseModel):
    message: str
    session_id: Optional[str] = None
```

**Usage Example (Python - `httpx` for SSE):**

```python
import httpx
import asyncio

async def stream_chat_response(user_message: str, session_id: str = None):
    url = "http://localhost:8000/chat"
    headers = {"Accept": "text/event-stream"}
    payload = {"message": user_message}
    if session_id:
        payload["session_id"] = session_id

    async with httpx.AsyncClient() as client:
        async with client.stream("POST", url, json=payload, headers=headers, timeout=None) as response:
            response.raise_for_status()
            print("Streaming chat response:")
            async for chunk in response.aiter_bytes():
                # Basic SSE parsing (adapt for robust parsing in production)
                decoded_chunk = chunk.decode('utf-8')
                if decoded_chunk.startswith("data:"):
                    data = decoded_chunk[len("data:"):].strip()
                    if data:
                        print(data, end='', flush=True)
            print("\n--- End of Stream ---")

# Example usage:
# asyncio.run(stream_chat_response("What is the capital of France?"))
# asyncio.run(stream_chat_response("Tell me more about it.", session_id="my_session_123"))
```

**Usage Example (JavaScript - `EventSource`):**

```javascript
const chatInput = document.getElementById('chatInput');
const sendButton = document.getElementById('sendButton');
const chatOutput = document.getElementById('chatOutput');
let eventSource = null;

sendButton.addEventListener('click', () => {
    const message = chatInput.value;
    if (!message) return;

    chatOutput.innerHTML += `<p><strong>You:</strong> ${message}</p>`;
    chatInput.value = '';

    // Close previous connection if open
    if (eventSource) {
        eventSource.close();
    }

    // Use fetch for POST request to initiate stream
    fetch('http://localhost:8000/chat', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Accept': 'text/event-stream' // Important for SSE
        },
        body: JSON.stringify({ message: message, session_id: "web_session_1" })
    })
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let buffer = '';

        chatOutput.innerHTML += `<p><strong>AI:</strong> <span id="aiResponse"></span></p>`;
        const aiResponseSpan = document.getElementById('aiResponse');

        function readStream() {
            reader.read().then(({ done, value }) => {
                if (done) {
                    console.log('Stream finished');
                    return;
                }
                buffer += decoder.decode(value, { stream: true });
                let lines = buffer.split('\n');
                buffer = lines.pop(); // Keep incomplete line in buffer

                lines.forEach(line => {
                    if (line.startsWith('data:')) {
                        const data = line.substring(5).trim();
                        if (data) {
                            aiResponseSpan.textContent += data;
                        }
                    }
                });
                readStream(); // Continue reading
            }).catch(error => {
                console.error('Stream error:', error);
                aiResponseSpan.textContent += ' (Error receiving response)';
            });
        }
        readStream();
    })
    .catch(error => {
        console.error('Fetch error:', error);
        chatOutput.innerHTML += `<p style="color:red;">Error: ${error.message}</p>`;
    });
});
```

#### 3. Generic Stream Generator

A generic endpoint for demonstrating server-sent events (SSE).

*   **Endpoint**: `GET /stream`
*   **Description**: Provides a continuous stream of messages, useful for testing SSE client implementations or for general-purpose real-time updates.
*   **Parameters**: None
*   **Returns**:
    *   `200 OK` with a `StreamingResponse` (Server-Sent Events - SSE). Each event will contain a simple message and a timestamp.

**Usage Example (Python - `httpx`):**

```python
import httpx
import asyncio

async def consume_stream():
    url = "http://localhost:8000/stream"
    headers = {"Accept": "text/event-stream"}

    async with httpx.AsyncClient() as client:
        async with client.stream("GET", url, headers=headers, timeout=None) as response:
            response.raise_for_status()
            print("Consuming stream:")
