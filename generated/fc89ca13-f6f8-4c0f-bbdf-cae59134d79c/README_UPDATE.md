## README Update: API Enhancements and New Features

This update introduces significant enhancements to the API, focusing on improved observability, expanded functionality, and better user experience.

### What's New

*   **Comprehensive Logging:** The API now includes extensive logging across various operations, providing better insights into application behavior and aiding in debugging.
*   **Health Check Endpoint:** A new `/health` endpoint has been added to monitor the API's operational status.
*   **Enhanced Document Upload:** The document upload functionality has been significantly improved to support the upload of **multiple files** in a single request.
*   **Streaming Chat Endpoint:** A new endpoint for streaming chat interactions has been introduced, enabling real-time communication.
*   **API Reset Functionality:** A new endpoint allows for resetting the API state, useful for development and testing.
*   **Improved API Documentation:** FastAPI application metadata has been updated for clearer and more comprehensive API documentation.

### How to Use New Features

*   **Health Check:**
    To check the API's health, simply make a GET request to the `/health` endpoint:
    ```bash
    GET /health
    ```
    A successful response (e.g., `200 OK`) indicates the API is operational.

*   **Multi-File Document Upload:**
    The document upload endpoint (`/upload_documents` or similar, depending on your API structure) now accepts multiple files. You can send multiple files in a single `multipart/form-data` request.
    ```python
    import requests

    files = [
        ('files', ('file1.txt', open('file1.txt', 'rb'), 'text/plain')),
        ('files', ('file2.pdf', open('file2.pdf', 'rb'), 'application/pdf'))
    ]
    response = requests.post('http://localhost:8000/upload_documents', files=files)
    print(response.json())
    ```
    *(Note: Replace `http://localhost:8000/upload_documents` with your actual endpoint and adjust file names/types as needed.)*

*   **Streaming Chat:**
    Details on how to interact with the streaming chat endpoint will be provided in the API documentation (accessible via `/docs` or `/redoc`). Typically, this will involve a WebSocket connection or Server-Sent Events (SSE).

*   **API Reset:**
    To reset the API state, make a POST request to the `/reset` endpoint:
    ```bash
    POST /reset
    ```
    *(Note: This endpoint should be used with caution, especially in production environments.)*

### Breaking Changes

There are no breaking changes introduced in this update. Existing API functionality should continue to work as before.

### Installation and Usage

No changes to the installation or basic usage instructions are required. The API can be started and used as previously documented.