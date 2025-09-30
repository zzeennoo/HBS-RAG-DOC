## README Update: API Refactor with Enhanced Features

This update introduces significant enhancements to the API, focusing on improved observability, new functionalities, and better management.

### What's New

*   **Comprehensive Logging:** The API now includes extensive logging for application events and HTTP requests, improving traceability and debugging capabilities.
*   **Health Check Endpoint:** A new `/health` endpoint has been added to monitor the API's operational status.
*   **Multi-File Uploads:** The `/upload` endpoint now supports uploading multiple documents simultaneously.
*   **Streaming Chat Responses:** The `/chat` endpoint now provides real-time, streaming responses for chat interactions.
*   **RAG Chain Reset:** A new `/reset` endpoint allows for clearing the vector store and resetting the RAG (Retrieval Augmented Generation) chain.
*   **Enhanced API Documentation:** The FastAPI documentation has been updated with a more descriptive title, description, and version.

### How to Use New Features

*   **Health Check:** Access `GET /health` to check the API's status.
*   **Multi-File Upload:** When uploading files to `/upload`, send a list of files in your request.
*   **Streaming Chat:** When interacting with `POST /chat`, your client should be configured to handle `text/event-stream` responses.
*   **Reset RAG Chain:** Send a `POST` request to `/reset` to clear the vector store and reset the RAG chain.

### Breaking Changes

Please be aware of the following breaking changes:

*   **`/upload` Endpoint:** The `/upload` endpoint now expects a `List[UploadFile]`. If you were previously sending a single `UploadFile`, you must now wrap it in a list (e.g., `[your_single_file]`).
*   **`/chat` Endpoint:** The `/chat` endpoint now returns a `StreamingResponse` by default. If your client is not configured to handle streaming responses, it will break. The previous non-streaming behavior of `/chat` has been moved to a new endpoint: `POST /chat_response`.

### Installation/Usage Instructions

No changes to the installation process are required. For usage, please refer to the updated API documentation available at `/docs` once the application is running.