```markdown
## [Unreleased]

### Added
- Comprehensive logging for application events and HTTP requests.
- Health check endpoint (`/health`).
- Streaming responses for chat interactions (`/chat`).
- Endpoint to reset the RAG chain and clear the vector store (`/reset`).
- Enhanced FastAPI documentation with `title`, `description`, and `version`.
- New non-streaming chat endpoint (`/chat_response`) for backward compatibility.

### Changed
- Enhanced upload functionality to support multiple files simultaneously.

### Fixed
- Improved API robustness and observability through refactoring.

### Breaking
- The `/upload` endpoint now expects a `List[UploadFile]`. Clients sending a single `UploadFile` without wrapping it in a list will need to update their requests.
- The `/chat` endpoint now returns a `StreamingResponse` by default. Clients expecting a non-streaming response should now use the new `/chat_response` endpoint.

_PR #4 by @zzeennoo_
```