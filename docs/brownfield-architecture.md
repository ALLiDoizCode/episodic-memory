# Episodic Memory Brownfield Architecture Document

## Introduction

This document captures the **CURRENT STATE** of the Episodic Memory codebase, a semantic search tool for Claude Code conversations. This is not an aspirational document - it reflects the actual implementation, patterns, and constraints as they exist today.

**Purpose**: This document serves as a reference for AI development agents working on enhancements, bug fixes, or refactoring. It documents what EXISTS, not what should exist.

### Document Scope

Comprehensive documentation of the entire Episodic Memory system, including:
- Core indexing and search engine
- CLI tools and commands
- MCP server integration
- Claude Code plugin
- Database schema and vector search
- Architecture patterns and technical constraints

### Change Log

| Date       | Version | Description                 | Author      |
| ---------- | ------- | --------------------------- | ----------- |
| 2025-11-07 | 1.0     | Initial brownfield analysis | John (PM)   |

## Quick Reference - Key Files and Entry Points

### Critical Files for Understanding the System

**Core Library (src/):**
- **Main Entry**: `src/index.ts` - Public API exports
- **Types**: `src/types.ts` - TypeScript interfaces for ConversationExchange, SearchResult, etc.
- **Indexer**: `src/indexer.ts` - Main indexing logic (batch processing, summarization)
- **Search**: `src/search.ts` - Vector and text search implementation
- **Database**: `src/db.ts` - SQLite schema, migrations, and data access
- **Embeddings**: `src/embeddings.ts` - Transformers.js embedding generation
- **Parser**: `src/parser.ts` - JSONL conversation file parser
- **Sync**: `src/sync.ts` - Atomic file syncing with conflict resolution
- **MCP Server**: `src/mcp-server.ts` - Model Context Protocol server

**CLI Tools (cli/):**
- `cli/episodic-memory.js` - Unified CLI entry point
- `cli/index-conversations.js` - Legacy indexing command
- `cli/search-conversations` - Legacy search command
- `cli/mcp-server` - MCP server launcher
- `cli/mcp-server-wrapper.js` - Plugin integration wrapper

**Claude Code Plugin:**
- `.claude-plugin/plugin.json` - Plugin manifest
- `agents/search-conversations.md` - Search agent definition
- `hooks/hooks.json` - Session-end hook configuration

**Testing:**
- `test/*.test.ts` - Vitest test suite

**Configuration:**
- `package.json` - Dependencies and build scripts
- `tsconfig.json` - TypeScript compilation config
- `vitest.config.ts` - Test runner config

## High Level Architecture

### Technical Summary

Episodic Memory is a **TypeScript-based semantic search engine** for Claude Code conversation histories. It uses **local offline embedding generation** (Transformers.js), **SQLite with vector search** (sqlite-vec), and **AI-powered summarization** to create a searchable memory system.

**Key Architectural Decisions:**
- **Offline-first**: Embeddings generated locally, no external API dependencies
- **SQLite**: Single-file database with WAL mode for concurrency
- **Atomic operations**: File syncing uses temp files + rename for consistency
- **Batch processing**: Concurrent summarization for performance
- **MCP integration**: Exposes search/read tools to Claude via Model Context Protocol

### Actual Tech Stack

| Category             | Technology               | Version      | Notes                                           |
| -------------------- | ------------------------ | ------------ | ----------------------------------------------- |
| Runtime              | Node.js                  | 18+          | ES Modules, requires modern Node                |
| Language             | TypeScript               | ^5.9.3       | Strict mode, ES2022 target                      |
| Package Manager      | npm                      | -            | Standard npm, no special config                 |
| Database             | better-sqlite3           | ^12.4.1      | Synchronous SQLite with native bindings         |
| Vector Search        | sqlite-vec               | ^0.1.7       | Vector similarity extension for SQLite          |
| Embeddings           | @xenova/transformers     | ^2.17.2      | Local Transformers.js (all-MiniLM-L6-v2 model)  |
| AI SDK               | @anthropic-ai/claude-agent-sdk | ^0.1.9 | Conversation summarization                     |
| MCP                  | @modelcontextprotocol/sdk| ^1.20.0      | Model Context Protocol server implementation    |
| Validation           | zod                      | ^3.25.76     | Input validation for MCP tools                  |
| Markdown             | marked                   | ^16.4.0      | Conversation display formatting                 |
| Build Tool           | esbuild                  | ^0.25.11     | Fast bundling for MCP server                    |
| Test Framework       | vitest                   | ^3.2.4       | Fast unit/integration tests                     |

### Repository Structure Reality Check

- **Type**: Single package (not monorepo)
- **Package Manager**: npm with package-lock.json
- **Module System**: ES Modules (type: "module")
- **Build Output**: `dist/` (TypeScript compilation) + bundled MCP server
- **Notable**: Plugin structure in `.claude-plugin/` for distribution

## Source Tree and Module Organization

### Project Structure (Actual)

```
episodic-memory/
├── src/                    # Core TypeScript source
│   ├── index.ts            # Public API exports
│   ├── types.ts            # TypeScript type definitions
│   ├── db.ts               # SQLite database layer
│   ├── embeddings.ts       # Embedding generation (Transformers.js)
│   ├── indexer.ts          # Main indexing orchestration
│   ├── parser.ts           # JSONL conversation parser
│   ├── search.ts           # Search implementation (vector + text)
│   ├── sync.ts             # File synchronization logic
│   ├── mcp-server.ts       # MCP server (bundled separately)
│   ├── summarizer.ts       # AI conversation summarization
│   ├── paths.ts            # Path utilities and configuration
│   ├── verify.ts           # Database verification tools
│   ├── stats.ts            # Index statistics
│   ├── show.ts             # Conversation display formatter
│   ├── *-cli.ts            # CLI command implementations
│
├── cli/                    # Executable CLI tools
│   ├── episodic-memory.js  # Main CLI entry (unified interface)
│   ├── index-conversations.js   # Legacy: indexing
│   ├── search-conversations     # Legacy: search
│   ├── mcp-server              # MCP server launcher
│   └── mcp-server-wrapper.js   # Plugin wrapper
│
├── test/                   # Vitest test suite
│   ├── *.test.ts           # Unit and integration tests
│   └── fixtures/           # Test data
│
├── dist/                   # Build output (TypeScript + esbuild)
│   ├── *.js                # Compiled JS modules
│   ├── *.d.ts              # Type declarations
│   └── mcp-server.js       # Bundled MCP server (esbuild)
│
├── .claude-plugin/         # Claude Code plugin configuration
│   ├── plugin.json         # Plugin manifest
│   └── marketplace.json    # Marketplace metadata
│
├── agents/                 # Agent definitions
│   └── search-conversations.md  # Search agent prompt
│
├── hooks/                  # Claude Code hooks
│   └── hooks.json          # Session-end indexing hook
│
├── docs/                   # Documentation
│   ├── SCHEMA.md           # Database schema reference
│   └── brownfield-architecture.md  # This file
│
├── commands/               # Slash commands (legacy?)
├── prompts/                # Prompt templates
├── scripts/                # Build/deployment scripts
├── skills/                 # Skill definitions
│
└── [Config Files]
    ├── package.json        # Dependencies, scripts, bin entries
    ├── tsconfig.json       # TypeScript compiler config
    ├── vitest.config.ts    # Test configuration
    ├── .gitignore
    └── LICENSE
```

### Key Modules and Their Purpose

**Core Indexing Pipeline:**
- `indexer.ts`: Orchestrates the entire indexing workflow - file discovery, parsing, summarization, embedding generation, database insertion
- `parser.ts`: Parses Claude Code JSONL conversation files into structured exchanges
- `embeddings.ts`: Generates 384-dim vector embeddings using Transformers.js (all-MiniLM-L6-v2)
- `summarizer.ts`: Generates AI summaries using Claude SDK (20k token limit)
- `db.ts`: SQLite database operations, schema migrations, exchange insertion

**Search System:**
- `search.ts`: Implements vector similarity search, text search, and multi-concept AND search
- `db.ts`: Vector search via sqlite-vec extension (vec0 virtual table)

**File Management:**
- `sync.ts`: Atomic file syncing from `~/.claude/projects` to archive with exclusion markers
- `paths.ts`: Centralized path management (archive dir, DB path, config paths)

**CLI Layer:**
- `*-cli.ts`: Individual command implementations (index, search, sync, stats, show)
- `cli/*.js`: Executable wrappers that call the CLI implementations

**Integration:**
- `mcp-server.ts`: MCP server exposing `search` and `read` tools (bundled with esbuild)
- `.claude-plugin/plugin.json`: Plugin configuration (agents, MCP servers, hooks)

## Data Models and APIs

### Data Models

**Primary types defined in `src/types.ts`:**

```typescript
// Core conversation exchange (user-agent pair)
interface ConversationExchange {
  id: string;                  // MD5 hash of archive_path:line_start-line_end
  project: string;             // Project name (from directory)
  timestamp: string;           // ISO timestamp
  userMessage: string;         // User's message text
  assistantMessage: string;    // Assistant's response (may be multi-part)
  archivePath: string;         // Path to archived JSONL file
  lineStart: number;           // Starting line number in JSONL
  lineEnd: number;             // Ending line number in JSONL

  // Optional metadata
  parentUuid?: string;         // Parent exchange (for sidechains)
  isSidechain?: boolean;       // True if subagent conversation
  sessionId?: string;          // Session UUID
  cwd?: string;                // Working directory
  gitBranch?: string;          // Git branch name
  claudeVersion?: string;      // Claude version
  thinkingLevel?: string;      // Thinking mode level
  thinkingDisabled?: boolean;  // Thinking disabled flag
  thinkingTriggers?: string;   // JSON array of thinking triggers
  toolCalls?: ToolCall[];      // Tool usage in this exchange
}

// Tool call tracking
interface ToolCall {
  id: string;
  exchangeId: string;
  toolName: string;
  toolInput?: any;
  toolResult?: string;
  isError: boolean;
  timestamp: string;
}

// Search results
interface SearchResult {
  exchange: ConversationExchange;
  similarity: number;          // 0-1 similarity score
  snippet: string;             // First 200 chars of user message
}

// Multi-concept search results
interface MultiConceptResult {
  exchange: ConversationExchange;
  snippet: string;
  conceptSimilarities: number[];  // Per-concept scores
  averageSimilarity: number;      // Average score
}
```

### Database Schema

See `docs/SCHEMA.md` for complete SQL schema. Key tables:

- **exchanges**: Main conversation data (see ConversationExchange interface)
- **tool_calls**: Tool usage tracking with foreign key to exchanges
- **vec_exchanges**: Virtual table (sqlite-vec) for vector embeddings (384-dim FLOAT[])

**Migrations**: Schema evolution handled via `migrateSchema()` in `db.ts` - checks for missing columns and adds them via ALTER TABLE. Idempotent and safe to run multiple times.

### Public API (src/index.ts exports)

```typescript
// Exported modules
export * from './types.js';
export * from './db.js';
export * from './embeddings.js';
export * from './indexer.js';
export * from './parser.js';
export * from './search.js';
export * from './summarizer.js';
export * from './paths.js';
```

**Key functions:**

**Indexing:**
- `indexConversations(limitToProject?, maxConversations?, concurrency?, noSummaries?)` - Full indexing
- `indexSession(sessionId, concurrency?, noSummaries?)` - Index single session
- `indexUnprocessed(concurrency?, noSummaries?)` - Incremental indexing

**Search:**
- `searchConversations(query, options?)` - Single-concept search (vector/text/both)
- `searchMultipleConcepts(concepts[], options?)` - Multi-concept AND search
- `formatResults(results)` - Format search results as markdown

**Database:**
- `initDatabase()` - Initialize/connect to SQLite (includes migrations)
- `insertExchange(db, exchange, embedding, toolNames?)` - Insert exchange with vector
- `getAllExchanges(db)` - Get all exchange IDs and paths
- `deleteExchange(db, id)` - Delete exchange (main + vector tables)

**Embeddings:**
- `initEmbeddings()` - Load Transformers.js model (lazy init)
- `generateEmbedding(text)` - Generate 384-dim vector
- `generateExchangeEmbedding(userMsg, assistantMsg, toolNames?)` - Combined embedding

**Parsing:**
- `parseConversation(filePath, projectName, archivePath)` - Parse JSONL file
- `parseConversationFile(filePath)` - Convenience wrapper

**Summarization:**
- `summarizeConversation(exchanges[])` - Generate AI summary using Claude SDK

**File Operations:**
- `syncConversations(sourceDir, destDir, options?)` - Atomic file sync + indexing

### CLI Commands

**Unified CLI** (`episodic-memory <command>`):
- `sync` - Sync and index new conversations (recommended for hooks)
- `index [--cleanup|--verify|--repair]` - Manual indexing operations
- `search <query> [--text|--after|--before|--limit]` - Search conversations
- `show <path> [--format html|markdown]` - Display conversation
- `stats` - Show index statistics

**Legacy commands** (still available):
- `episodic-memory-index` - Index conversations
- `episodic-memory-search <query>` - Search conversations

### MCP Tools

**search** tool:
- Single-concept: `{ query: "string", mode: "vector"|"text"|"both", limit: 1-50 }`
- Multi-concept: `{ query: ["concept1", "concept2"], limit: 1-50 }`
- Optional: `after`, `before` (YYYY-MM-DD), `response_format` (markdown|json)

**read** tool:
- `{ path: "/absolute/path.jsonl", startLine?: 1, endLine?: 100 }`
- Line numbers are 1-indexed, inclusive

## Technical Debt and Known Issues

### Critical Technical Debt

1. **Embedding Model Fixed**: Uses all-MiniLM-L6-v2 (384-dim) hardcoded in `embeddings.ts`. Changing models would require re-indexing entire database. No migration path for model upgrades.

2. **No Database Versioning**: While schema migrations exist, there's no database version tracking. If schema changes incompatibly, detection is manual.

3. **Summarization API Dependency**: Conversation summarization requires Claude API access via `@anthropic-ai/claude-agent-sdk`. No fallback for offline operation or API failures. Can be skipped with `--no-summaries` flag.

4. **Tool Result Matching**: `parser.ts` has TODO comment: "Match tool_use_id to previous tool_use" for tool results. Currently tool results are parsed but not linked back to tool calls.

5. **Single Database File**: All conversations share one SQLite file (`~/.claude/conversation-search/conversations.db`). No sharding or multi-database support for large-scale deployments.

6. **Native Dependencies**: `better-sqlite3` requires native compilation. Postinstall script attempts rebuild but can fail on some platforms. Windows support recently fixed (v1.0.8).

### Workarounds and Gotchas

**Environment Variables:**
- `TEST_PROJECTS_DIR`: Override projects directory for testing
- `CONVERSATION_SEARCH_EXCLUDE_PROJECTS`: Comma-separated list of projects to skip
- `CLAUDE_CODE_MAX_OUTPUT_TOKENS`: Set to 20000 in indexer for summarization

**Exclusion Markers**: Conversations with these markers are archived but not indexed:
- `<INSTRUCTIONS-TO-EPISODIC-MEMORY>DO NOT INDEX THIS CHAT</INSTRUCTIONS-TO-EPISODIC-MEMORY>`
- `Only use NO_INSIGHTS_FOUND` (summary prompt indicator)
- Summary context instructions (avoid meta-conversations)

**WAL Mode**: Database uses Write-Ahead Logging for concurrency. Means additional `-wal` and `-shm` files alongside `.db` file.

**Embedding Truncation**: Text truncated to 2000 chars before embedding (model limit: 512 tokens). Long conversations may lose context.

**Postinstall Hook**: `npm rebuild better-sqlite3` runs on install, errors suppressed with `|| true`. May silently fail.

**MCP Server Bundling**: MCP server uses esbuild with extensive externals list (fsevents, sharp, onnxruntime-node, etc.) to avoid bundling native modules. Changes to dependencies may require updating externals list.

### Performance Considerations

**Batch Concurrency**: Summarization uses configurable concurrency (default: 1) to avoid API rate limits. Indexing is fast (embeddings are local), but summarization is slow.

**First Run**: Initial embedding model download (Xenova/all-MiniLM-L6-v2) takes time. Model cached in `~/.cache/huggingface/` afterward.

**Vector Search Speed**: sqlite-vec is fast for typical conversation counts (hundreds to thousands). Performance with 10k+ conversations not tested.

## Integration Points and External Dependencies

### External Services

| Service        | Purpose               | Integration Type | Key Files                       |
| -------------- | --------------------- | ---------------- | ------------------------------- |
| Claude API     | Conversation summarization | REST API (via SDK) | `src/summarizer.ts`        |
| Hugging Face   | Embedding model download | HTTP download | `src/embeddings.ts` (Transformers.js) |

**Notes:**
- Claude API requires `ANTHROPIC_API_KEY` environment variable for summarization
- Hugging Face models downloaded to `~/.cache/huggingface/` (no auth required)
- Both are **optional** - indexing works without summaries (`--no-summaries`)

### Internal Integration Points

**Claude Code Plugin Integration:**
- **MCP Server**: Registered in `.claude-plugin/plugin.json` as `episodic-memory`
- **Session Hooks**: `hooks/hooks.json` triggers `episodic-memory sync` at session end
- **Agents**: `agents/search-conversations.md` defines natural language search interface
- **Command**: MCP server started via `cli/mcp-server-wrapper.js` (sets plugin root env var)

**File System Integration:**
- **Source**: Reads from `~/.claude/projects/{project}/*.jsonl`
- **Archive**: Copies to `~/.claude/conversation-search/archive/{project}/*.jsonl`
- **Database**: `~/.claude/conversation-search/conversations.db`
- **Summaries**: `~/.claude/conversation-search/archive/{project}/*-summary.txt`
- **Exclusions**: `~/.claude/conversation-search/exclude-projects.txt`

**Data Flow:**
```
~/.claude/projects/{project}/*.jsonl
    ↓ (sync.ts - atomic copy)
~/.claude/conversation-search/archive/{project}/*.jsonl
    ↓ (parser.ts)
ConversationExchange[]
    ↓ (summarizer.ts - optional)
Summary text → *-summary.txt
    ↓ (embeddings.ts)
384-dim vectors
    ↓ (db.ts)
SQLite (exchanges + vec_exchanges tables)
    ↓ (search.ts)
Search results
```

## Development and Deployment

### Local Development Setup

**Prerequisites:**
- Node.js 18+ (ES Modules support required)
- npm (comes with Node)
- Python (for better-sqlite3 native compilation on first install)

**Setup steps:**
```bash
# Clone repository
git clone https://github.com/obra/episodic-memory.git
cd episodic-memory

# Install dependencies (includes postinstall rebuild)
npm install

# Run tests
npm test

# Build
npm run build
```

**Build output:**
- TypeScript compilation: `dist/*.js`, `dist/*.d.ts`
- MCP server bundle: `dist/mcp-server.js` (esbuild)

**Development workflow:**
```bash
# Watch tests during development
npm run test:watch

# Manual testing with local CLI
node cli/episodic-memory.js --help
node cli/episodic-memory.js sync
node cli/episodic-memory.js search "your query"
```

### Build and Deployment Process

**Build Command**: `npm run build`
1. `tsc` - Compile TypeScript to `dist/`
2. `npm run bundle` - Bundle MCP server with esbuild

**Package.json bin entries:**
```json
"bin": {
  "episodic-memory": "./cli/episodic-memory.js",
  "episodic-memory-index": "./cli/index-conversations.js",
  "episodic-memory-search": "./cli/search-conversations",
  "episodic-memory-mcp-server": "./cli/mcp-server"
}
```

**Deployment as npm package:**
```bash
# Publish to npm (maintainer only)
npm version patch  # or minor/major
npm publish

# Users install via
npm install episodic-memory

# Or as Claude Code plugin
/plugin install episodic-memory@superpowers-marketplace
```

**Plugin Distribution:**
- Marketplace: Published to superpowers-marketplace
- Manifest: `.claude-plugin/plugin.json` + `marketplace.json`
- Plugin includes: compiled code, agents, hooks, CLI binaries

### Testing Reality

**Test Framework**: Vitest (fast, TypeScript native)

**Current Test Coverage:**
- `test/db.test.ts` - Database operations
- `test/parser.test.ts` - JSONL parsing
- `test/sync.test.ts` - File synchronization
- `test/multi-concept.test.ts` - Multi-concept search
- `test/search-agent-template.test.ts` - Agent template validation

**Test Configuration:**
- Timeout: 30 seconds (for embedding/indexing operations)
- Environment: Node
- Globals: Enabled (describe, it, expect)

**Running Tests:**
```bash
npm test           # Run all tests once
npm run test:watch # Watch mode for development
```

**Coverage Gaps:**
- No integration tests for full indexing pipeline
- No MCP server tests (requires MCP client mock)
- No CLI command tests
- Embeddings/summarization tested manually (external dependencies)

## Coding Standards and Conventions

### Language and Style

**TypeScript Strict Mode:**
- `strict: true` in tsconfig.json
- Explicit type annotations for public APIs
- `any` used sparingly (mostly for tool inputs)

**Module System:**
- ES Modules exclusively (`type: "module"`)
- All imports use `.js` extension (TypeScript requires this for ESM)
- Named exports preferred over default exports

**Naming Conventions:**
- Files: kebab-case (`conversation-parser.ts`)
- Functions: camelCase (`parseConversation`)
- Classes: PascalCase (minimal use, mostly functions)
- Interfaces: PascalCase (`ConversationExchange`)
- Constants: UPPER_SNAKE_CASE for config (`EXCLUSION_MARKERS`)

### Code Organization Patterns

**Single Responsibility**: Each module has one clear purpose (db.ts = database, embeddings.ts = embedding generation)

**Dependency Injection**: Database instance passed to functions, not global singleton

**Error Handling**: Try-catch with graceful degradation (e.g., skip malformed JSONL lines, continue on summary failures)

**Async Patterns**: Heavy use of async/await, Promise.all for parallelization

**Configuration**: Environment variables + config files (exclude-projects.txt)

### File Conventions

**JSONL Format**: Claude Code conversation files are newline-delimited JSON
- Each line is a JSON object with `type`, `message`, `timestamp`, etc.
- Parser handles malformed lines gracefully (continue, not fail)

**Summary Files**: Plain text, stored adjacent to JSONL with `-summary.txt` suffix

**Archive Structure**: Mirrors source structure (`~/.claude/projects/{project}/*.jsonl` → `archive/{project}/*.jsonl`)

## Appendix - Useful Commands and Scripts

### Frequently Used Commands

**Indexing:**
```bash
# Full reindex of all conversations
episodic-memory index

# Index only unprocessed conversations (incremental)
episodic-memory index --cleanup

# Index with concurrency (10 parallel summaries)
episodic-memory-index --concurrency 10

# Skip AI summaries (faster, offline-friendly)
episodic-memory index --no-summaries
```

**Searching:**
```bash
# Semantic + text search (default)
episodic-memory search "React Router authentication"

# Vector-only search
episodic-memory search "database schema" --mode vector

# Search with date filters
episodic-memory search "refactoring" --after 2025-09-01 --before 2025-10-01

# Limit results
episodic-memory search "deployment" --limit 5

# Text-only search (exact matching)
episodic-memory search "exact phrase" --text
```

**Syncing:**
```bash
# Recommended for session-end hook
episodic-memory sync

# Sync without indexing (copy files only)
episodic-memory sync --skip-index

# Sync without summaries (faster)
episodic-memory sync --skip-summaries
```

**Display:**
```bash
# Show full conversation as markdown
episodic-memory show path/to/conversation.jsonl

# Show as HTML
episodic-memory show --format html conversation.jsonl > output.html
open output.html

# Show specific line range (for large conversations)
episodic-memory show conversation.jsonl --start 1 --end 50
```

**Statistics:**
```bash
# View index statistics
episodic-memory stats
```

### Debugging and Troubleshooting

**Common Issues:**

1. **"better-sqlite3 not found" error**:
   ```bash
   npm rebuild better-sqlite3
   ```

2. **Slow first run**: Embedding model download (normal, one-time)
   ```bash
   # Model cached in ~/.cache/huggingface/
   ls -lh ~/.cache/huggingface/transformers
   ```

3. **Exclude project from indexing**:
   ```bash
   # Add to exclude file
   echo "my-project-name" >> ~/.claude/conversation-search/exclude-projects.txt

   # Or use env var
   export CONVERSATION_SEARCH_EXCLUDE_PROJECTS="project1,project2"
   ```

4. **Database corruption**:
   ```bash
   # Verify database
   episodic-memory index --verify

   # Repair if needed
   episodic-memory index --repair

   # Nuclear option: delete and reindex
   rm ~/.claude/conversation-search/conversations.db*
   episodic-memory index
   ```

5. **API key for summaries**:
   ```bash
   # Set Claude API key
   export ANTHROPIC_API_KEY="your-key"

   # Or skip summaries
   episodic-memory index --no-summaries
   ```

**Debug Mode:**
- MCP server logs to stderr: `console.error('...')` visible in Claude Code logs
- CLI output: `console.log()` goes to stdout
- No built-in debug flag (add console.log as needed)

**Inspecting Database:**
```bash
# Open SQLite database directly
sqlite3 ~/.claude/conversation-search/conversations.db

# Useful queries:
sqlite> SELECT COUNT(*) FROM exchanges;
sqlite> SELECT project, COUNT(*) FROM exchanges GROUP BY project;
sqlite> SELECT * FROM exchanges ORDER BY timestamp DESC LIMIT 5;
sqlite> .tables
sqlite> .schema exchanges
```

### Package Scripts

```bash
npm run build        # Full build (TypeScript + esbuild)
npm run bundle       # Bundle MCP server only
npm test             # Run test suite
npm run test:watch   # Watch mode for tests
```

### Environment Variables Reference

| Variable                              | Purpose                           | Default                                      |
| ------------------------------------- | --------------------------------- | -------------------------------------------- |
| `ANTHROPIC_API_KEY`                   | Claude API for summarization      | None (required for summaries)                |
| `TEST_PROJECTS_DIR`                   | Override projects dir for tests   | `~/.claude/projects`                         |
| `CONVERSATION_SEARCH_EXCLUDE_PROJECTS`| Projects to skip (comma-separated)| None (or from exclude-projects.txt)          |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS`       | Max tokens for summarization      | 20000 (set in indexer.ts)                    |
| `CLAUDE_PLUGIN_ROOT`                  | Plugin root for MCP wrapper       | Set by Claude Code plugin system             |

---

**Document Status**: Complete
**Last Updated**: 2025-11-07
**Maintainer**: Product Management / AI Development Team
