# CLAUDE.md - AI Assistant Guide for Claude Planning Tool

> **Last Updated:** 2025-11-30
> **Project Status:** Planning Phase (~25% complete)
> **Primary Language:** Finnish (documentation), English (code)

---

## Project Overview

**Claude Planning Tool** is an AI-assisted software planning tool designed for long-running software development sessions. It leverages Claude's 1M token context window with external memory persistence via Markdown files and SQLite vector search.

### Core Goals

1. **1M Token Context Window** - Full utilization of Claude Sonnet 4.5's extended context
2. **External Memory** - Persistent storage in human-readable Markdown files
3. **Semantic Search** - Vector-based memory retrieval using SQLite-vec and Ollama embeddings
4. **Session Management** - Context-aware planning across multiple sessions

### Tech Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Backend | Python 3.11+ / FastAPI | Async API server |
| AI | Claude API (Sonnet 4.5) | 1M context, planning assistance |
| Database | SQLite + sqlite-vec | Metadata + vector search |
| Embeddings | Ollama (nomic-embed-text) | 768-dim vectors, local/free |
| Storage | Markdown files | Human-readable, Git-friendly |

---

## Directory Structure

```
claude-planning-tool/
├── CLAUDE.md                    # This file - AI assistant guide
├── README.md                    # Project overview (Finnish)
├── KEHITYSLOKI.md              # Development log / session history
├── PROCESS_SPEC_Writing.md     # Spec writing process guide
├── LICENSE
├── .gitignore
│
├── docs/
│   ├── specs/                  # Functional specifications
│   │   ├── SPEC_01_Claude_Service.md      # ✅ v1.2 Final
│   │   ├── SPEC_02_Memory_Service.md      # ✅ v1.1 Review Complete
│   │   └── SPEC_03_Context_Manager.md     # ✅ v1.1 Gemini-reviewed
│   │
│   ├── tech-specs/             # Technical implementation specs
│   │   └── TECH_SPEC_02_Memory_Service.md # ✅ v1.0 Approved
│   │
│   └── research/               # Research documents
│       ├── RESEARCH_01_Claude_API_Patterns.md
│       ├── RESEARCH_02_Memory_Architecture.md
│       ├── RESEARCH_03_Context_Manager.md
│       └── TECH_RESEARCH_02_Memory_Service.md
│
├── guides/
│   └── GUIDE_Extended_Thinking_Usage.md  # When to use extended thinking
│
└── src/                        # (Not yet created - planned structure)
    └── memory/
        ├── service.py          # MemoryService facade
        ├── models.py           # Pydantic data models
        ├── repository.py       # SQLite operations
        ├── file_store.py       # Markdown file handling
        ├── embedding_client.py # Ollama integration
        └── text_splitter.py    # Text chunking
```

---

## Development Workflow

### Document-First Approach

This project follows a **Research → Spec → Tech Spec → Implementation** workflow:

```
1. RESEARCH_XX.md    →  Research phase, gather best practices
2. SPEC_XX.md        →  Functional specification (what to build)
3. TECH_SPEC_XX.md   →  Technical implementation (how to build)
4. src/              →  Implementation code
```

### Spec Writing Process (PROCESS_SPEC_Writing.md)

1. **Context** - Read architecture docs and previous specs
2. **Research** - Web search for best practices, save findings
3. **Planning** - Identify primitives, API design, structure
4. **Writing** - Write the SPEC document
5. **User Review** - Owner reviews and approves
6. **Gemini Review** - Technical QA with Google's Gemini
7. **Finalization** - Apply corrections, save final version

### Versioning Convention

Documents follow semantic versioning:
- `v1.0` - Initial version
- `v1.1` - Minor updates (often from Gemini review)
- `v1.2` - Additional refinements

Status markers:
- `🔲` - Not started
- `🚧` - In progress
- `✅` - Complete/Approved

---

## Core Modules

### 1. ClaudeService (SPEC_01)
**Status:** ✅ Final v1.2

Black-box wrapper for Claude API. All API calls go through this service.

Key features:
- `send_message()` / `stream_message()` / `count_tokens()`
- Circuit breaker pattern for resilience
- Prompt caching support (90% cost savings)
- Token usage tracking via `MetricsTracker`
- Proactive token warnings (80%, 90%, 95% thresholds)

### 2. MemoryService (SPEC_02, TECH_SPEC_02)
**Status:** ✅ Review Complete

External memory management with hybrid architecture.

Key features:
- Markdown files for human-readable storage
- SQLite-vec for semantic search
- Parent-Child chunking (512 tokens, 50 overlap)
- Category-based salience decay (decisions never decay)
- Graceful degradation when Ollama unavailable

Categories:
- `decisions` - Architecture decisions (salience decay: 1.0 = never)
- `summaries` - Session summaries (decay: 0.99 = slow)
- `context` - Working memory (decay: 0.90 = fast)

### 3. ContextManager (SPEC_03)
**Status:** ✅ v1.1 Gemini-reviewed

Context building and token budget management.

Key features:
- Priority-based content selection (MUST/SHOULD/NICE)
- Token budget management (200K default, 20K reserve)
- Prompt caching strategy (SYSTEM_ONLY for MVP)
- Security: prompt injection detection and sanitization
- XML-structured context with clear data/instruction separation

---

## Key Conventions

### Documentation Language
- **Documentation:** Finnish (primary audience)
- **Code/APIs:** English
- **This file:** English (for AI assistants)

### Code Style
- Python 3.11+ with type hints
- Pydantic v2 for data validation
- Async/await throughout (aiosqlite, aiofiles, httpx)
- Raw SQL preferred over ORM for sqlite-vec compatibility

### Security Guidelines

From SPEC_01 and SPEC_03:

```python
# ❌ NEVER log sensitive data
logger.info(f"Request: {messages}")    # Leaks user data
logger.debug(f"API key: {config.api_key}")  # Leaks secrets

# ✅ Log only metadata
logger.info(f"Request: model={model}, tokens={token_count}")
```

Prompt injection protection:
- Escape XML in user content
- Wrap file content with security warnings
- Separate trusted instructions from untrusted data
- Validate input patterns (see `INJECTION_PATTERNS` in SPEC_03)

### Error Handling

Standard patterns:
- Exponential backoff with jitter for retries
- Circuit breaker for cascade prevention
- Graceful degradation (save without embeddings if Ollama down)
- Never silently fail on critical operations

---

## AI Assistant Guidelines

### When Working on This Project

1. **Read specs first** - Check `docs/specs/` before implementing
2. **Follow the primitives** - Each module has a defined primitive (Message, MemoryItem, ContextBlock)
3. **Respect black-box boundaries** - Modules communicate through clean APIs only
4. **Update KEHITYSLOKI.md** - Log significant changes to the development log
5. **Use Finnish for docs** - Unless creating English-only content like CLAUDE.md

### Extended Thinking Usage (from GUIDE)

| Situation | Extended Thinking | Reason |
|-----------|-------------------|--------|
| Research + web search | OFF | Timeout risk with many tool calls |
| Document writing | OFF | Creative work, not analysis |
| Architecture decisions | ON | Benefits from systematic analysis |
| Code debugging | ON | Step-by-step reasoning helps |
| Long sessions (>30 messages) | OFF + new window | Cumulative timeout risk |

**Rule of thumb:**
- "Search and write" → OFF
- "Analyze and decide" → ON
- If unsure → OFF (safer)

### Gemini Review Integration

Specs are reviewed by Google's Gemini for technical QA. Common review patterns:

- Configuration injection (avoid hardcoded values)
- Missing interface definitions
- Security considerations
- Edge case handling
- Category-based behavior differences

---

## Current Project Status

```
[███░░░░░░░] 25% - Planning (SPEC layer)
```

### Module Status

| Module | RESEARCH | SPEC | TECH_SPEC | Code |
|--------|----------|------|-----------|------|
| ClaudeService | ✅ v1.0 | ✅ v1.2 | 🔲 | 🔲 |
| MemoryService | ✅ v1.0 | ✅ v1.1 | ✅ v1.0 | 🔲 |
| ContextManager | ✅ v1.0 | ✅ v1.1 | 🔲 | 🔲 |

### Next Steps

1. TECH_SPEC_01 for ClaudeService
2. TECH_SPEC_03 for ContextManager
3. Begin MVP implementation

---

## Quick Reference

### API Patterns

```python
# ClaudeService
async def send_message(messages, system_prompt=None, ...) -> Message
async def stream_message(messages, ...) -> AsyncGenerator[StreamEvent]
def count_tokens(messages, ...) -> TokenCount  # FREE API call!

# MemoryService
async def save(title, content, category) -> MemoryItem
async def search(query, limit=5) -> List[SearchResult]
async def get_context_for_prompt(query, max_tokens) -> List[ContextBlock]

# ContextManager
async def build_context(user_message, ...) -> ContextResult
def get_token_usage() -> TokenUsage
def validate_input(text) -> ValidationResult
```

### Important Constants

```python
# Token thresholds
TOKEN_THRESHOLDS = {
    "premium_zone": 0.2,   # 200K/1M - higher pricing starts
    "warning": 0.8,        # 80% - suggest summarization
    "critical": 0.9,       # 90% - recommend action
    "danger": 0.95,        # 95% - urgent
}

# Context budget (MVP)
MAX_CONTEXT = 200_000      # tokens
RESERVE_FOR_RESPONSE = 20_000  # tokens

# Embedding
EMBEDDING_DIMENSIONS = 768  # nomic-embed-text
CHUNK_SIZE = 512           # tokens
CHUNK_OVERLAP = 50         # tokens
```

---

## Contributing

1. Follow the existing document structure
2. Use Finnish for documentation, English for code
3. Update KEHITYSLOKI.md with session summaries
4. Get Gemini review for spec changes
5. Save intermediate work frequently (timeout protection)

---

*This file provides context for AI assistants working on the Claude Planning Tool project.*
