# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RAG (Retrieval-Augmented Generation) chatbot for querying DeepLearning.AI course materials. Full-stack app with a Python/FastAPI backend and vanilla HTML/JS/CSS frontend. Uses Claude API with tool calling for search, ChromaDB for vector storage, and Sentence Transformers for embeddings.

## Commands

```bash
# Install dependencies
uv sync

# Run the app (from project root)
cd backend && uv run uvicorn app:app --reload --port 8000

# Alternative: use the provided script
./run.sh
```

Access at `http://localhost:8000`. Swagger API docs at `http://localhost:8000/docs`.

## Environment Setup

Requires Python 3.13+ and `uv`. Copy `.env.example` to `.env` and set `ANTHROPIC_API_KEY`.

## Architecture

**Backend (`backend/`):**
- `app.py` — FastAPI entry point. Serves frontend static files and exposes `/api/query` (POST) and `/api/courses` (GET). Loads course docs from `../docs/` on startup.
- `rag_system.py` — Main orchestrator. Chains document processing → vector search → Claude generation. Entry point is `query()`.
- `document_processor.py` — Parses structured `.txt` course files into metadata and sentence-based chunks (800 chars, 100 overlap).
- `vector_store.py` — ChromaDB wrapper with two collections: `course_catalog` (metadata) and `course_content` (text chunks). Uses `all-MiniLM-L6-v2` embeddings.
- `ai_generator.py` — Claude API client (`claude-sonnet-4-20250514`, temperature=0). Handles tool calling loop for search.
- `search_tools.py` — Defines `CourseSearchTool` for Claude tool use. ToolManager registers/executes tools and tracks sources.
- `session_manager.py` — Per-session conversation history (max 2 turns).
- `config.py` — Centralized settings. Key values: chunk size 800, overlap 100, max 5 search results.
- `models.py` — Pydantic models for requests/responses.

**Frontend (`frontend/`):**
- Static chat UI with dark theme, course stats sidebar, suggested questions, markdown rendering (marked.js), and source attribution display.

**Data (`docs/`):**
- Course material `.txt` files with structured format: `Course Title:`, `Course Instructor:`, `Lesson N:` markers.

## Data Flow

User query → FastAPI `/api/query` → RAG system gets/creates session → sends query + history to Claude with search tool → Claude optionally calls `search_course_content` → vector store returns relevant chunks → Claude generates final response → response + sources returned to frontend.

## Key Conventions

- **Always use `uv` to run the server, execute Python files, and manage dependencies. Do not use `pip` or bare `python` directly — use `uv run` instead.**
- ChromaDB persists to `./chroma_db/` (gitignored)
- No test suite exists currently
- Frontend is served by FastAPI as static files (relative path `../frontend`)
- All config is in `backend/config.py`, not scattered across files

## UI Guidelines

- Always make any buttons for making noise the color pink
