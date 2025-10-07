This document outlines the API changes introduced in the "Refactor API with logging, streaming, and management features" pull request. This update introduces new logging, streaming, and system management capabilities, alongside an enhanced file upload mechanism and a new health check endpoint.

---

## API Documentation: Refactor API with Logging, Streaming, and Management Features

### 1. New Endpoints and Functions

This section details the newly added API endpoints, including their purpose, parameters, and expected return values.

#### 1.1. Health Check

A simple endpoint to verify the API's operational status.

*   **Endpoint:** `GET /health`
*   **Function:** `health_check()`
*   **Description:** Returns a success message if the API is running.
*   **Parameters:** None
*   **Returns:**
    *   `200 OK` with a JSON body:
        ```json
        {
            "status": "healthy",
            "message": "API is up and running!"
        }
        ```
*   **Example Usage (cURL):**
    ```bash
    curl -X GET "http://localhost:8000/health"
    ```
*   **Example Response:**
    ```json
    {
        "status": "healthy",
        "message": "API is up and running!"
    }
    ```

#### 1.2. Chat Stream

Initiates a streaming chat session, allowing for real-time interaction.

*   **Endpoint:** `POST /chat/stream`
*   **Function:** `chat_stream(request: ChatRequest)`
*   **Description:** Establishes a server-sent event (SSE) stream for real-time chat responses. The client sends a `ChatRequest` and receives a continuous stream of chat messages.
*   **Parameters:**
    *   `request` (Body - `ChatRequest`):
        *   `message` (string, required): The user's message to the chat model.
        *   `session_id` (string, optional): An identifier for the chat session. If not provided, a new session might be initiated or a default one used.
*   **Returns:**
    *   `200 OK` with `Content-Type: text/event-stream`. The response will be a continuous stream of server-sent events, each containing a piece of the chat response.
*   **Example Usage (cURL):**
    ```bash
    curl -X POST "http://localhost:8000/chat/stream" \
         -H "Content-Type: application/json" \
         -d '{
               "message": "Tell me about the latest advancements in AI.",
               "session_id": "user123_session456"
             }'
    ```
*   **Example Response (Stream):**
    ```
    data: {"type": "chunk", "content": "The latest advancements"}
    data: {"type": "chunk", "content": " in AI include large"}
    data: {"type": "chunk", "content": " language models,"}
    data: {"type": "chunk", "content": " generative AI,"}
    data: {"type": "chunk", "content": " and improved reinforcement"}
    data: {"type": "chunk", "content": " learning techniques."}
    data: {"type": "end"}
    ```
*   **Error Handling:**
    *   `422 Unprocessable Entity`: If the `ChatRequest` body is malformed or missing required fields.

#### 1.3. System Reset

Resets the system to its initial state, potentially clearing data or configurations.

*   **Endpoint:** `POST /system/reset`
*   **Function:** `reset_system()`
*   **Description:** Performs a system-wide reset. This operation is typically destructive and should be used with caution. The exact scope of the reset (e.g., clearing all uploaded documents, resetting chat history, reinitializing models) depends on the backend implementation.
*   **Parameters:** None
*   **Returns:**
    *   `200 OK` with a JSON body:
        ```json
        {
            "status": "success",
            "message": "System has been reset successfully."
        }
        ```
*   **Example Usage (cURL):**
    ```bash
    curl -X POST "http://localhost:8000/system/reset"
    ```
*   **Example Response:**
    ```json
    {
        "status": "success",
        "message": "System has been reset successfully."
    }
    ```
*   **Error Handling:**
    *   `500 Internal Server Error`: If the reset operation fails due to an internal issue.

#### 1.4. Internal Stream Generator (Middleware/Helper)

*   **Function:** `stream_generator()`
*   **Description:** This is an internal asynchronous generator function likely used by other streaming endpoints (like `/chat/stream`) to produce data chunks. It's not a directly exposed API endpoint but a core component for server-sent event (SSE) streaming.
*   **Parameters:** None (or internal parameters passed during its invocation)
*   **Returns:** An `AsyncGenerator` yielding string chunks.
*   **Note:** This function is typically not called directly by external clients.

#### 1.5. Request Logging Middleware

*   **Function:** `log_requests(request: Request, call_next)`
*   **Description:** This is an asynchronous middleware function that intercepts all incoming requests and outgoing responses. It logs details about each request (e.g., method, path, client IP) and the corresponding response (e.g., status code, duration). This is crucial for monitoring, debugging, and auditing API usage.
*   **Parameters:**
    *   `request` (`Request`): The incoming FastAPI request object.
    *   `call_next` (callable): A function that proceeds to the next middleware or the actual endpoint handler.
*   **Returns:** The `Response` object generated by the downstream handler.
*   **Note:** This is a backend middleware and does not represent an accessible API endpoint. Its presence ensures that all API interactions are logged.

---

### 2. Modified Endpoints

This section details changes to existing API endpoints.

#### 2.1. Document Upload

The document upload functionality has been enhanced to support multiple file uploads simultaneously.

*   **Endpoint:** `POST /upload/documents`
*   **Old Function:** `upload_document(file: UploadFile = File(...))`
*   **New Function:** `upload_documents(files: List[UploadFile] = File(...))`
*   **Description of Change:** The endpoint now accepts a list of `UploadFile` objects instead of a single `UploadFile`. This allows clients to upload multiple documents in a single request, improving efficiency for batch operations.
*   **Parameters:**
    *   `files` (Form Data - `List[UploadFile]`, required): A list of files to be uploaded. Each `UploadFile` object contains the file content, filename, and content type.
*   **Returns:**
    *   `200 OK` with a JSON body:
        ```json
        {
            "status": "success",
            "uploaded_count": 2,
            "message": "Successfully uploaded 2 documents.",
            "details": [
                {"filename": "document1.pdf", "size_bytes": 12345},
                {"filename": "image.png", "size_bytes": 67890}
            ]
        }
        ```
*   **Example Usage (cURL):**
    ```bash
    curl -X POST "http://localhost:8000/upload/documents" \
         -H "accept: application/json" \
         -H "Content-Type: multipart/form-data" \
         -F "files=@/path/to/your/document1.pdf" \
         -F "files=@/path/to/your/image.png"
    ```
*   **Example Response:**
    ```json
    {
        "status": "success",
        "uploaded_count": 2,
        "message": "Successfully uploaded 2 documents.",
        "details": [
            {"filename": "document1.pdf", "size_bytes": 12345},
            {"filename": "image.png", "size_bytes": 67890}
        ]
    }
    ```
*   **Error Handling:**
    *   `422 Unprocessable Entity`: If no files are provided or if the request body is malformed.
    *   `500 Internal Server Error`: If there's an issue processing or storing one or more files. The response might include details about which files failed.

---

### 3. Breaking Changes

The `upload_document` endpoint has been removed and replaced by `upload_documents`.

*