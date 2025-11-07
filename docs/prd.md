# Episodic Memory - AO/Arweave Decentralized Memory Enhancement PRD

## Executive Summary

**Enhancement:** Add cross-device agent continuity, multi-agent shared memory, and hierarchical knowledge search through AO/Arweave decentralized backend + Permamind skill registry, orchestrated via existing MCP servers (permaweb-mcp, permamind-mcp).

**Key Architectural Decision:** Leverage MCP orchestration pattern instead of building custom AO/Arweave integration from scratch. Episodic-memory MCP server calls permaweb-mcp and permamind-mcp tools, reducing implementation complexity by ~70%.

**Value Proposition:**
- ✅ **Cross-Device Continuity:** Agent remembers context when user switches devices
- ✅ **Multi-Agent Collaboration:** Dev/PM/QA agents share knowledge base
- ✅ **Hierarchical Memory:** Search local project first, fall back to global skills registry
- ✅ **Knowledge Publishing:** Valuable conversations become reusable skills for other projects
- ✅ **Permanent Archive:** Conversations stored immutably on Arweave
- ✅ **Backward Compatible:** Local SQLite continues working (AO/Arweave opt-in)
- ✅ **Autonomous Behavior:** Agent automatically searches memory via MCP tools (no user commands)

**Implementation Approach:**
- **Hybrid Backend:** SQLite cache + AO metadata + Arweave content
- **Hierarchical Search:** Local episodic-memory → Global permamind registry
- **MCP Orchestration:** Use permaweb-mcp tools (uploadToArweave, spawnProcess, sendAOMessage, readAOProcess)
- **Skill Synergy:** Permamind-mcp for cross-project knowledge sharing (CORE MVP)
- **Stories:** 18 total (17 MVP + 1 polish)

**Memory Hierarchy:**
```
Layer 1: Current Conversation (immediate context)
  ↓
Layer 2: Episodic Memory (THIS project's history)
  ↓
Layer 3: Permamind Skills (OTHER projects' patterns)
  ↓
Layer 4: Base Knowledge (Claude's training)
```

---

## Intro Project Analysis and Context

### Scope Assessment

This PRD addresses a **significant enhancement** to the Episodic Memory project that requires comprehensive planning and multiple coordinated development stories. This is NOT a simple feature addition - it represents a fundamental architectural expansion to enable cross-device agent continuity and multi-agent collaboration through decentralized storage.

**Scope Justification:** This enhancement involves:
- Adding new backend infrastructure (AO process + Arweave integration)
- Maintaining backward compatibility with existing SQLite implementation
- Implementing intelligent caching and synchronization logic
- Creating autonomous agent memory behavior
- Complex integration across multiple systems (local, AO, Arweave)

This requires full PRD process with multiple stories, careful sequencing, and architectural planning.

### Project Context

**Analysis Source:** IDE-based fresh analysis + document-project output

**Document-Project Output Available:** Yes - `docs/brownfield-architecture.md` created 2025-11-07

### Existing Project Overview

#### Analysis Source

**Source:** Document-project analysis completed 2025-11-07
**Location:** `docs/brownfield-architecture.md`

#### Current Project State

**Project:** Episodic Memory v1.0.9

**Current Purpose:** Semantic search engine for Claude Code conversation histories. Enables agents to search past conversations using vector similarity (local embeddings via Transformers.js) and text search against a local SQLite database.

**Key Capabilities:**
- Parse Claude Code conversation JSONL files
- Generate 384-dim vector embeddings locally (offline, no API calls)
- Store in SQLite with sqlite-vec for fast vector similarity search
- AI-powered conversation summarization (Claude API)
- MCP server exposing `search` and `read` tools to Claude
- CLI tools for indexing, searching, and syncing
- Claude Code plugin with session-end hooks for automatic indexing

**Current Limitations:**
- **Single-device only:** Database stuck on one machine, no cross-device memory
- **Isolated per user:** Each user's agent has completely separate memory
- **Local-only storage:** If database deleted, all conversation history lost
- **No collaboration:** Multiple agents cannot share knowledge base

### Available Documentation Analysis

#### Available Documentation

**Document-Project Analysis Available:** Yes

**Key Documents Created by document-project:**
- ✅ `docs/brownfield-architecture.md` - Complete technical architecture
- ✅ `docs/SCHEMA.md` - Database schema reference (existing)
- ✅ Source code analysis (all modules in `src/`)

#### Documentation Coverage

**From document-project output:**

✅ **Tech Stack Documentation:**
- Runtime: Node.js 18+ (ES Modules)
- Language: TypeScript ^5.9.3 (strict mode)
- Database: better-sqlite3 ^12.4.1 + sqlite-vec ^0.1.7
- Embeddings: @xenova/transformers ^2.17.2 (all-MiniLM-L6-v2)
- AI SDK: @anthropic-ai/claude-agent-sdk ^0.1.9
- MCP: @modelcontextprotocol/sdk ^1.20.0

✅ **Source Tree/Architecture:**
- Core modules: indexer, search, db, embeddings, parser, sync, mcp-server
- CLI tools: unified CLI + legacy commands
- Plugin structure: `.claude-plugin/` with agents, hooks, MCP config
- Test suite: Vitest with 30s timeout for embedding tests

✅ **Coding Standards:**
- TypeScript strict mode, ES2022 target
- ES Modules with `.js` extensions
- Named exports, camelCase functions, PascalCase interfaces
- Error handling with graceful degradation

✅ **API Documentation:**
- Public API exported from `src/index.ts`
- MCP tools: `search` (single/multi-concept), `read` (full conversation)
- CLI commands: sync, index, search, show, stats

✅ **External API Documentation:**
- Claude API for summarization (optional, can use --no-summaries)
- Hugging Face for embedding model download (cached locally)

✅ **Technical Debt Documentation:**
- Embedding model locked to all-MiniLM-L6-v2 (384-dim)
- No database version tracking
- Tool result matching incomplete (TODO in parser.ts)
- Single database file (no sharding)
- Native dependencies (better-sqlite3 compilation issues)

### Enhancement Scope Definition

#### Enhancement Type

- ☑ New Feature Addition
- ☐ Major Feature Modification
- ☑ Integration with New Systems
- ☐ Performance/Scalability Improvements
- ☐ UI/UX Overhaul
- ☐ Technology Stack Upgrade
- ☐ Bug Fix and Stability Improvements

**Primary Type:** New Feature Addition (AO/Arweave backend) + Integration with New Systems (blockchain infrastructure)

#### Enhancement Description

Add decentralized backend infrastructure to Episodic Memory using AO (Actor Oriented) processes and Arweave permanent storage, enabling **cross-device agent continuity** and **multi-agent shared memory** while maintaining backward compatibility with local SQLite operation.

**Key Additions:**
1. **AO Process Integration:** Lua-based metadata index for fast conversation queries
2. **Arweave Storage:** Permanent conversation archive (JSONL + summaries)
3. **Hybrid Architecture:** SQLite becomes intelligent cache layer when AO/Arweave enabled
4. **Autonomous Memory:** MCP tools transparently query AO/Arweave, agent decides when to search
5. **Cross-Device Sync:** Agent remembers context across devices automatically
6. **Multi-Agent Collaboration:** Shared knowledge base for dev/PM/QA agents

#### Impact Assessment

- ☐ Minimal Impact (isolated additions)
- ☐ Moderate Impact (some existing code changes)
- ☑ Significant Impact (substantial existing code changes)
- ☐ Major Impact (architectural changes required)

**Rationale:** While core modules (parser, embeddings, search logic) remain mostly unchanged, significant changes required for:
- MCP server backend abstraction (swap SQLite for AO/Arweave queries)
- Database layer (add caching logic, dual-backend support)
- Sync workflow (upload to Arweave + index to AO)
- Configuration system (wallet management, process IDs, backend selection)
- New modules for AO communication and Arweave interaction

### Goals and Background Context

#### Goals

**Enhancement Goals:**

1. **Enable Cross-Device Agent Continuity:** Agent maintains full conversation context when user switches between devices (desktop → laptop → mobile)

2. **Enable Multi-Agent Shared Memory:** Multiple agents (dev, PM, QA) can query and learn from shared knowledge base

3. **Provide Permanent Conversation Archive:** Conversations stored immutably on Arweave, surviving any local data loss

4. **Maintain Backward Compatibility:** Existing users continue using local SQLite without configuration; AO/Arweave is opt-in enhancement

5. **Preserve Performance:** Search remains fast through intelligent SQLite caching; cross-device sync happens in background

6. **Autonomous Agent Behavior:** Agent automatically searches memory when relevant, no manual user commands required

#### Background Context

**Why This Enhancement is Needed:**

Current episodic-memory implementation is **excellent for single-device, single-user scenarios** but has fundamental limitations for distributed agent collaboration:

**Problem 1: Device Fragmentation**
- User works on desktop Monday, laptop Tuesday, tablet Wednesday
- Each device has isolated conversation history in local SQLite
- Agent on laptop has zero context from desktop sessions
- User must manually re-explain decisions, repeat context
- **Result:** Agent effectiveness degrades across devices

**Problem 2: Agent Isolation**
- Dev agent learns about authentication implementation
- PM agent (different session) asks about same topic
- PM agent has no access to dev agent's knowledge
- Both agents re-discover same solutions independently
- **Result:** Duplicated effort, inconsistent knowledge

**Problem 3: Data Permanence**
- Conversation history lives in `~/.claude/conversation-search/`
- Disk failure, accidental deletion, computer loss → all context gone
- No backup strategy, no recovery path
- **Result:** Months/years of valuable agent-learned context at risk

**How This Fits with Existing Project:**

Episodic Memory already has the **right architecture for decentralization**:
- ✅ MCP server provides tool interface (swap backend transparently)
- ✅ Modular design separates parsing, search, storage layers
- ✅ Conversation format is immutable JSONL (perfect for Arweave)
- ✅ Metadata-heavy search (already separates content from index)
- ✅ Agent-driven workflows (search tools, not user commands)

This enhancement **leverages existing strengths** while adding distributed infrastructure.

**User Value:**
- Work from anywhere, agent always has full context
- Team collaboration through shared agent memory
- Never lose conversation history (permanent Arweave storage)
- No workflow changes (still automatic via hooks + MCP tools)

### Change Log

| Change | Date | Version | Description | Author |
|--------|------|---------|-------------|---------|
| Initial | 2025-11-07 | 1.0 | Initial brownfield PRD creation | John (PM) |

---

## Requirements

### Functional Requirements

**FR1:** System SHALL support hybrid backend architecture with local SQLite and optional AO/Arweave distributed storage

**FR2:** When AO/Arweave backend is NOT configured, system SHALL operate identically to current local-only SQLite implementation (backward compatibility)

**FR3:** When AO/Arweave backend IS configured, system SHALL upload new conversations to Arweave and index metadata in AO process during sync operations

**FR4:** System SHALL use SQLite as intelligent cache layer when AO/Arweave enabled, checking cache before querying distributed backend

**FR5:** MCP server search tool SHALL transparently query appropriate backend (local SQLite or AO/Arweave) based on configuration without changing tool interface

**FR6:** MCP server read tool SHALL fetch full conversations from Arweave when backend is AO/Arweave, caching results locally in SQLite

**FR7:** System SHALL support cross-device synchronization by querying AO process for conversation metadata and fetching content from Arweave

**FR8:** System SHALL enable multi-agent shared memory by allowing multiple agents to query same AO process and access same Arweave content

**FR9:** Session-end hook SHALL automatically sync new conversations to configured backend (local SQLite or AO/Arweave) without user intervention

**FR10:** Claude agent SHALL autonomously search conversation memory when relevant using MCP tools, without requiring manual user commands

**FR11:** System SHALL maintain vector similarity search performance through local embedding generation (Transformers.js) and SQLite cache hits

**FR12:** System SHALL provide configuration interface for setting AO process ID, Arweave wallet, and cache behavior

**FR13:** System SHALL handle offline scenarios gracefully by falling back to SQLite cache when AO/Arweave unreachable

**FR14:** System SHALL preserve conversation immutability on Arweave (JSONL files never modified after upload)

**FR15:** AO process SHALL index conversation metadata including: ID, project, timestamp, Arweave transaction IDs, tool names, tags, session context

**FR16:** MCP search tool SHALL implement hierarchical search strategy: (1) search local episodic-memory, (2) if results insufficient, search permamind registry, (3) clearly indicate result source

**FR17:** System SHALL provide `publish-skill` command to extract insights from conversations and publish to permamind registry using `mcp__permamind__publish_skill` tool

**FR18:** Published skills SHALL include: conversation summary, code snippets, decision rationale, tags (topic, tools, language), and reference to original conversation

**FR19:** Hierarchical search results SHALL distinguish between local project memory and global skill registry with clear source indicators

**FR20:** Agent SHALL autonomously search permamind registry when local episodic-memory returns insufficient results (threshold: <3 results)

### Non-Functional Requirements

**NFR1: Performance - Search Speed**
- Local SQLite search: <10ms (current baseline)
- SQLite cache hit with AO/Arweave: <10ms (no degradation)
- AO metadata query: <200ms
- Arweave content fetch: <1000ms per conversation
- Multi-concept search with cache: <50ms

**NFR2: Performance - Sync Operations**
- Arweave upload: <5s per conversation (including summary)
- AO indexing: <500ms per conversation
- Batch sync (100 conversations): <10 minutes with concurrency
- Background sync SHALL NOT block user interaction

**NFR3: Reliability - Graceful Degradation**
- When offline: Fall back to SQLite cache (cached conversations searchable)
- When AO process unreachable: Fall back to local SQLite
- When Arweave gateway slow: Cache results for retry
- System SHALL log errors but continue operation

**NFR4: Reliability - Data Integrity**
- Arweave uploads SHALL be atomic (temp file + rename pattern)
- AO process updates SHALL handle concurrent writes from multiple devices
- SQLite cache SHALL support WAL mode for concurrent reads
- No data loss during sync failures (local SQLite retains copy)

**NFR5: Scalability - Conversation Volume**
- AO process SHALL efficiently handle 10,000+ conversation metadata entries
- SQLite cache SHALL support configurable size limits (default: recent 7 days or 100 conversations)
- Arweave SHALL handle unlimited conversation storage (pay-per-byte)

**NFR6: Scalability - Multi-Agent Access**
- AO process SHALL support concurrent queries from 10+ agents simultaneously
- AO message handlers SHALL process requests in <100ms under load
- Arweave gateway SHALL handle parallel fetches (Promise.all for batch downloads)

**NFR7: Security - Data Privacy**
- Arweave uploads SHALL support optional encryption (future enhancement)
- AO process SHALL support access control via wallet signatures (future enhancement)
- Local SQLite cache SHALL remain private (no sharing without user configuration)
- Wallet private keys SHALL NEVER be stored in code or logged

**NFR8: Usability - Configuration**
- Initial setup (wallet + AO process ID) SHALL complete in <5 minutes
- Configuration errors SHALL provide clear, actionable error messages
- System SHALL provide configuration validation before first sync
- Default configuration SHALL work for 80% of users (local-only)

**NFR9: Usability - Autonomy**
- Agent SHALL search memory automatically when user question references past work
- No manual `/search` or `/read` commands required
- MCP tool descriptions SHALL guide agent to use tools proactively
- User SHALL NOT need to understand AO/Arweave to benefit from features

**NFR10: Cost Efficiency**
- Arweave storage cost: <$0.01 per typical conversation (10KB)
- AO message costs: <$0.001 per query
- SQLite cache SHALL reduce redundant Arweave fetches by >80%
- Batch uploads SHALL minimize transaction overhead

**NFR11: Maintainability - Code Quality**
- Backend abstraction SHALL isolate SQLite vs AO/Arweave implementation details
- Shared interfaces for search, index, sync operations
- Dependency injection for database/backend instances (no global singletons)
- Comprehensive error handling with specific error types

**NFR12: Testability**
- Unit tests for AO message handlers (mock AO SDK)
- Integration tests for Arweave upload/download (test wallet)
- MCP server tests SHALL work with both backends
- Cache invalidation tests (time-based expiry, manual clear)

### Compatibility Requirements

**CR1: API Compatibility**
- MCP tool interface SHALL remain unchanged (`search` and `read` tools with same schemas)
- Public API exports from `src/index.ts` SHALL maintain backward compatibility
- CLI commands SHALL support same flags and arguments (add new, don't break existing)

**CR2: Database Schema Compatibility**
- SQLite schema SHALL support existing tables (exchanges, tool_calls, vec_exchanges)
- Add cache metadata columns (last_cached, arweave_tx, cache_ttl) via migration
- Schema migrations SHALL be idempotent and backward compatible

**CR3: UI/UX Consistency**
- Agent behavior SHALL remain autonomous (no new manual commands required)
- Search results format SHALL match current markdown output
- Configuration commands SHALL follow existing CLI patterns (`episodic-memory config set ...`)

**CR4: Integration Compatibility**
- Claude Code plugin hooks SHALL continue working (session-end triggers sync)
- Agent definitions in `agents/` SHALL work with both backends
- Existing test suite SHALL pass with local SQLite backend (no regressions)

**CR5: File Format Compatibility**
- Conversation JSONL files SHALL use identical format (no breaking changes)
- Summary text files SHALL maintain same `-summary.txt` naming convention
- Archive directory structure SHALL remain compatible (`archive/{project}/*.jsonl`)

**CR6: Configuration Compatibility**
- Environment variables (TEST_PROJECTS_DIR, CONVERSATION_SEARCH_EXCLUDE_PROJECTS) SHALL continue working
- Exclusion markers SHALL apply to AO/Arweave uploads (same filtering logic)
- Existing config files SHALL migrate gracefully (add new fields, keep existing)

---

## User Interface Enhancement Goals

**Note:** This enhancement does NOT include UI changes - the interface remains the autonomous MCP tool-driven workflow. This section captures how the invisible UX improves.

### Integration with Existing UI

**Current UX Pattern:** Agent autonomously searches memory via MCP tools when relevant

**Enhanced UX Pattern:** Same autonomous behavior, but now works across devices and agents

**Key Improvements:**
1. **Cross-Device Continuity:** User switches devices, agent seamlessly accesses same memory
2. **Multi-Agent Collaboration:** Different agents (dev, PM, QA) reference each other's learned context
3. **No New Commands:** User workflow unchanged (still automatic via hooks and MCP tools)

### Modified/New Screens and Views

**No visual screens added** (CLI tool, no GUI)

**CLI Output Changes:**

**New:** `episodic-memory config setup` - Interactive wizard for AO/Arweave configuration
```
Welcome to Episodic Memory AO/Arweave Setup!

This enables cross-device memory and multi-agent collaboration.

Step 1/3: AO Process
  ○ Use existing AO process
  ● Create new AO process

  Creating AO process... ✓
  Process ID: p_abc123...def789

Step 2/3: Arweave Wallet
  ○ Use existing wallet
  ● Generate new wallet

  Wallet address: xyz789...abc456

Step 3/3: Initial Sync
  Upload existing conversations to AO/Arweave? [Y/n]

  Uploading 150 conversations...
  [████████████████████████████████] 150/150

  ✅ Setup complete! Your agent now has cross-device memory.
```

**Enhanced:** `episodic-memory sync` - Shows backend status
```
Current Backend: AO/Arweave (hybrid mode)
AO Process: p_abc123...def789
Cache: 87 conversations (5.2MB), hit rate: 91%

Indexing new conversations...
  ✓ Parsed 3 new conversations
  ✓ Uploaded to Arweave (TX: abc123..., def456..., ghi789...)
  ✓ Indexed to AO process
  ✓ Cached locally

✅ Sync complete! 3 new conversations indexed.
```

**Enhanced:** `episodic-memory search` - Indicates backend source
```
Searching... (AO/Arweave backend)

Found 5 relevant conversations:

1. [my-app, 2025-11-05] - 94% match
   "Discussed JWT authentication with httpOnly cookies..."
   Tools: Read(3), Edit(5), Bash(2)
   Source: Cache (instant) ⚡

2. [my-app, 2025-10-28] - 87% match
   "Implemented token refresh rotation..."
   Tools: Edit(7), Bash(1)
   Source: Arweave (fetched in 450ms) 🌐
```

### UI Consistency Requirements

**Consistency with Existing Patterns:**
- Search results maintain same markdown format
- Error messages follow same style (clear, actionable)
- Progress indicators use same emoji/symbol conventions
- Configuration commands follow `episodic-memory config <action> <key> <value>` pattern

**Accessibility:**
- CLI output remains screen-reader friendly (plain text)
- No color dependencies (use symbols: ✓ ✗ ⚡ 🌐)
- Progress bars optional (can run with `--quiet` flag)

---

## Technical Constraints and Integration Requirements

### Existing Technology Stack

**From document-project output (`docs/brownfield-architecture.md`):**

| Category | Technology | Version | Notes |
|----------|------------|---------|-------|
| Runtime | Node.js | 18+ | ES Modules required |
| Language | TypeScript | ^5.9.3 | Strict mode, ES2022 |
| Database | better-sqlite3 | ^12.4.1 | Native compilation |
| Vector Search | sqlite-vec | ^0.1.7-alpha.2 | SQLite extension |
| Embeddings | @xenova/transformers | ^2.17.2 | all-MiniLM-L6-v2 model |
| AI SDK | @anthropic-ai/claude-agent-sdk | ^0.1.9 | Summarization |
| MCP | @modelcontextprotocol/sdk | ^1.20.0 | Tool interface |
| Validation | zod | ^3.25.76 | Input validation |
| Markdown | marked | ^16.4.0 | Formatting |
| Build | esbuild | ^0.25.11 | MCP server bundling |
| Test | vitest | ^3.2.4 | 30s timeout |

**NEW MCP Servers (Plugin Dependencies):**

| MCP Server | Package | Purpose |
|------------|---------|---------|
| permaweb-mcp | permaweb-mcp | AO process operations, Arweave uploads |
| permamind-mcp | @permamind/mcp | Skill registry (optional synergy) |

**NEW Code Dependencies:**

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| Claude SDK | @anthropic-ai/claude-agent-sdk | ^0.1.9 | MCP tool orchestration (already exists) |

**Removed Dependencies (handled by MCP servers):**
- ~~@permaweb/aoconnect~~ → Replaced by permaweb-mcp tools
- ~~arweave SDK~~ → Replaced by permaweb-mcp tools
- ~~@permaweb/wallet-kit~~ → Handled by permaweb-mcp (SEED_PHRASE)

### Integration Approach

#### MCP Server Orchestration Strategy

**Architecture Pattern:** Episodic-memory MCP server orchestrates permaweb-mcp and permamind-mcp tools instead of implementing AO/Arweave integration from scratch.

**Plugin Configuration (.claude-plugin/plugin.json):**

```json
{
  "name": "episodic-memory",
  "version": "2.0.0",
  "description": "Semantic search with cross-device memory via AO/Arweave",

  "mcpServers": {
    "episodic-memory": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/cli/mcp-server-wrapper.js"],
      "env": {}
    },
    "permaweb": {
      "command": "npx",
      "args": ["permaweb-mcp"],
      "env": {
        "SEED_PHRASE": "${SEED_PHRASE}"
      }
    },
    "permamind": {
      "command": "npx",
      "args": ["@permamind/mcp@latest"],
      "env": {
        "SEED_PHRASE": "${SEED_PHRASE}"
      }
    }
  },

  "hooks": "./hooks/hooks.json"
}
```

**MCP Tool Orchestration Pattern:**

```typescript
// src/backend/ao-arweave.ts
import { ClaudeCodeSDK } from '@anthropic-ai/claude-agent-sdk';

const claude = new ClaudeCodeSDK();

export class AOArweaveBackend implements SearchBackend {
  async uploadConversation(filePath: string): Promise<string> {
    // Use permaweb-mcp instead of custom Arweave SDK
    const result = await claude.callTool('mcp__permaweb__uploadToArweave', {
      filePath: filePath,
      paymentMethod: 'tokens'
    });

    // Parse transaction ID from result
    const txId = this.extractTxId(result.content[0].text);
    return txId;
  }

  async indexToAO(processId: string, conversation: ConversationExchange, arweaveTx: string): Promise<void> {
    // Use permaweb-mcp instead of custom aoconnect integration
    await claude.callTool('mcp__permaweb__sendAOMessage', {
      processId: processId,
      tags: [
        { name: 'Action', value: 'IndexConversation' },
        { name: 'ConversationId', value: conversation.id },
        { name: 'Project', value: conversation.project },
        { name: 'Timestamp', value: conversation.timestamp }
      ],
      data: JSON.stringify({
        ...conversation,
        arweave_conversation: arweaveTx,
        snippet: conversation.userMessage.substring(0, 200)
      })
    });
  }

  async searchAO(processId: string, query: string, filters: SearchFilters): Promise<any[]> {
    // Use permaweb-mcp read tool instead of custom client
    const result = await claude.callTool('mcp__permaweb__readAOProcess', {
      processId: processId,
      tags: [
        { name: 'Action', value: 'SearchConversations' },
        { name: 'Query', value: query },
        { name: 'Project', value: filters.project || '' },
        { name: 'Limit', value: String(filters.limit || 10) }
      ]
    });

    // Parse AO process response
    return JSON.parse(result.content[0].text);
  }

  async fetchFromArweave(txId: string): Promise<string> {
    // permaweb-mcp doesn't have a fetch tool, use HTTP gateway
    const response = await fetch(`https://arweave.net/${txId}`);
    return await response.text();
  }
}
```

**Benefits of MCP Orchestration:**
1. **Less Code:** Stories 1.3-1.5 become thin wrappers (~50-100 lines each vs 200-500 lines)
2. **Leverage Existing Tools:** permaweb-mcp already handles wallet management, transaction signing, AO message formatting
3. **Maintainability:** Upstream fixes in permaweb-mcp benefit episodic-memory automatically
4. **Separation of Concerns:** Episodic-memory focuses on search logic, permaweb-mcp handles blockchain operations

**Tradeoffs:**
1. **External Dependency:** If permaweb-mcp has issues, episodic-memory affected (mitigation: fallback to SQLite cache)
2. **Missing Tools:** permaweb-mcp currently lacks Arweave fetch/download (mitigation: simple HTTP gateway fetch)
3. **Abstraction Overhead:** Tool call latency vs direct SDK calls (negligible: ~1-5ms)

#### Database Integration Strategy

**Current SQLite Role:** Primary database with vec_exchanges virtual table for vector search

**New Hybrid Role:**

**1. Local-Only Mode (AO/Arweave NOT configured):**
```typescript
// No changes - current implementation
async function search(query: string) {
  const db = initDatabase();
  const results = await searchConversations(db, query);
  return results;
}
```

**2. Hybrid Mode (AO/Arweave configured):**
```typescript
// src/backend/router.ts
async function search(query: string, options: SearchOptions) {
  const config = loadConfig();

  if (!config.aoProcessId) {
    // Local-only mode
    return searchLocal(query, options);
  }

  // Check SQLite cache first
  const cached = await checkCache(query, options);
  if (cached && isCacheFresh(cached)) {
    return cached.results;
  }

  // Query AO for metadata
  const aoResults = await queryAO(config.aoProcessId, query);

  // Fetch content from Arweave (parallel)
  const conversations = await fetchFromArweave(aoResults);

  // Update SQLite cache
  await updateCache(query, conversations);

  return conversations;
}
```

**Schema Changes:**
```sql
-- Add cache metadata to existing exchanges table
ALTER TABLE exchanges ADD COLUMN arweave_tx TEXT;
ALTER TABLE exchanges ADD COLUMN last_cached INTEGER;
ALTER TABLE exchanges ADD COLUMN cache_ttl INTEGER DEFAULT 604800000; -- 7 days

-- Add backend config table
CREATE TABLE IF NOT EXISTS backend_config (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Add sync status tracking
CREATE TABLE IF NOT EXISTS sync_status (
  conversation_id TEXT PRIMARY KEY,
  synced_to_ao BOOLEAN DEFAULT 0,
  synced_to_arweave BOOLEAN DEFAULT 0,
  arweave_tx TEXT,
  ao_indexed_at INTEGER,
  last_sync_attempt INTEGER,
  sync_error TEXT
);
```

#### API Integration Strategy

**MCP Tool Interface (Unchanged):**
```typescript
// MCP tools maintain same schema
{
  name: 'search',
  inputSchema: {
    query: string | string[],
    mode: 'vector' | 'text' | 'both',
    limit: number,
    after?: string,
    before?: string
  }
}
```

**Backend Abstraction:**
```typescript
// src/backend/interface.ts
interface SearchBackend {
  search(query: string, options: SearchOptions): Promise<SearchResult[]>;
  index(exchange: ConversationExchange): Promise<void>;
  sync(): Promise<SyncResult>;
}

// src/backend/local.ts
class LocalSQLiteBackend implements SearchBackend {
  // Current implementation
}

// src/backend/ao-arweave.ts
class AOArweaveBackend implements SearchBackend {
  async search(query: string, options: SearchOptions): Promise<SearchResult[]> {
    // Query AO → Fetch Arweave → Cache SQLite
  }

  async index(exchange: ConversationExchange): Promise<void> {
    // Upload to Arweave → Index to AO → Cache SQLite
  }

  async sync(): Promise<SyncResult> {
    // Batch upload/index
  }
}

// src/backend/factory.ts
function createBackend(config: Config): SearchBackend {
  if (config.aoProcessId && config.wallet) {
    return new AOArweaveBackend(config);
  }
  return new LocalSQLiteBackend(config);
}
```

#### Frontend Integration Strategy

**N/A** - This is a CLI tool with no frontend. MCP server is the "frontend" interface for Claude.

**MCP Server Changes:**
```typescript
// src/mcp-server.ts
import { createBackend } from './backend/factory.js';

const backend = createBackend(loadConfig());

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'search') {
    // Backend abstraction - works with local or AO/Arweave
    const results = await backend.search(
      request.params.arguments.query,
      request.params.arguments
    );

    return { content: [{ type: 'text', text: formatResults(results) }] };
  }
});
```

#### Testing Integration Strategy

**Existing Test Suite:** Vitest with fixtures in `test/`

**New Test Coverage:**

```typescript
// test/backend/ao-arweave.test.ts
describe('AOArweaveBackend', () => {
  it('uploads conversation to Arweave', async () => {
    const mockArweave = createMockArweave();
    const backend = new AOArweaveBackend({
      arweave: mockArweave,
      wallet: testWallet
    });

    const tx = await backend.uploadConversation(testConversation);
    expect(tx).toMatch(/^[a-zA-Z0-9_-]{43}$/);
  });

  it('indexes metadata to AO process', async () => {
    const mockAO = createMockAO();
    const backend = new AOArweaveBackend({
      ao: mockAO,
      processId: testProcessId
    });

    await backend.indexToAO(testExchange, 'TX_ABC123');
    expect(mockAO.sentMessages).toHaveLength(1);
    expect(mockAO.sentMessages[0].tags).toContainEqual({
      name: 'Action',
      value: 'IndexConversation'
    });
  });

  it('falls back to SQLite when AO unreachable', async () => {
    const backend = new AOArweaveBackend({
      ao: createFailingMockAO()
    });

    const results = await backend.search('test query');
    expect(results).toBeDefined(); // From SQLite cache
  });
});

// test/backend/hybrid.test.ts
describe('Hybrid mode', () => {
  it('returns cached results for repeated searches', async () => {
    const backend = createBackend(hybridConfig);

    const first = await backend.search('authentication');
    const firstDuration = measureTime();

    const second = await backend.search('authentication');
    const secondDuration = measureTime();

    expect(secondDuration).toBeLessThan(firstDuration * 0.1); // Cache 10x faster
  });
});
```

**Test Fixtures:**
```
test/fixtures/
├── conversations/           # Sample JSONL files
├── ao-responses/            # Mock AO message results
├── arweave-transactions/    # Mock Arweave TX data
└── wallets/                 # Test wallet keys (funded on testnet)
```

### Code Organization and Standards

**From document-project (`docs/brownfield-architecture.md`):**

**Existing Patterns:**
- ES Modules with `.js` extensions in imports
- Named exports (no default exports)
- camelCase functions, PascalCase interfaces
- Strict TypeScript mode
- Error handling with try-catch and graceful degradation

**New Module Organization:**
```
src/
├── backend/                 # NEW: Backend abstraction
│   ├── interface.ts         # SearchBackend interface
│   ├── local.ts             # LocalSQLiteBackend implementation
│   ├── ao-arweave.ts        # AOArweaveBackend implementation
│   ├── router.ts            # Backend selection logic
│   └── factory.ts           # createBackend factory
│
├── ao/                      # NEW: AO-specific logic
│   ├── client.ts            # AO message sending/receiving
│   ├── handlers.lua         # Lua handlers for AO process
│   ├── spawn.ts             # AO process spawning
│   └── types.ts             # AO message types
│
├── arweave/                 # NEW: Arweave-specific logic
│   ├── client.ts            # Arweave transaction creation
│   ├── upload.ts            # File upload logic
│   ├── download.ts          # Transaction data fetching
│   └── wallet.ts            # Wallet management
│
├── cache/                   # NEW: SQLite cache management
│   ├── manager.ts           # Cache CRUD operations
│   ├── invalidation.ts      # TTL and LRU logic
│   └── types.ts             # Cache metadata types
│
├── config/                  # NEW: Configuration management
│   ├── manager.ts           # Config load/save
│   ├── validation.ts        # Config validation (zod schemas)
│   └── wizard.ts            # Interactive setup
│
├── [existing modules]       # db, embeddings, indexer, parser, search, etc.
│   └── [minimal changes]    # Add backend parameter injection
│
└── mcp-server.ts            # MODIFIED: Use backend abstraction
```

**File Naming:**
- Backend implementations: `{name}-backend.ts` (e.g., `local-backend.ts`)
- AO modules: `ao/{purpose}.ts`
- Arweave modules: `arweave/{purpose}.ts`

**Import Patterns:**
```typescript
// Always use .js extension for ESM
import { createBackend } from './backend/factory.js';
import { AOClient } from './ao/client.js';
import { uploadToArweave } from './arweave/upload.js';
```

**Error Handling:**
```typescript
// Specific error types
class AOConnectionError extends Error {
  constructor(message: string, public processId: string) {
    super(message);
    this.name = 'AOConnectionError';
  }
}

class ArweaveUploadError extends Error {
  constructor(message: string, public filePath: string, public cause?: Error) {
    super(message);
    this.name = 'ArweaveUploadError';
  }
}

// Graceful degradation
try {
  return await backend.searchAO(query);
} catch (error) {
  if (error instanceof AOConnectionError) {
    console.warn(`AO unreachable: ${error.message}, falling back to cache`);
    return await backend.searchCache(query);
  }
  throw error;
}
```

### Deployment and Operations

**From document-project (`docs/brownfield-architecture.md`):**

**Current Build Process:**
```bash
npm run build     # tsc + esbuild bundle
npm test          # vitest
```

**Enhanced Build for AO/Arweave:**
```bash
npm run build:ao-handlers   # Bundle Lua handlers for AO
npm run deploy:ao-process   # Deploy AO process to testnet/mainnet
```

**Build Script Changes:**
```json
// package.json
{
  "scripts": {
    "build": "tsc && npm run bundle && npm run build:ao-handlers",
    "bundle": "esbuild src/mcp-server.ts --bundle ...",
    "build:ao-handlers": "cat src/ao/handlers/*.lua > dist/ao-process.lua",
    "deploy:ao-process": "node scripts/deploy-ao-process.js",
    "test": "vitest run",
    "test:integration": "vitest run --config vitest.integration.config.ts"
  }
}
```

**Deployment Strategy:**

**Local Development:**
1. `npm install` - Installs all dependencies (including new AO/Arweave SDKs)
2. `npm run build` - Compiles TypeScript + bundles MCP server + Lua handlers
3. `episodic-memory config setup` - Interactive configuration (optional)
4. `episodic-memory sync` - Works locally or with AO/Arweave

**Production (npm package):**
1. User runs: `npm install -g episodic-memory@2.0.0`
2. First run detects no config → uses local SQLite (backward compatible)
3. User optionally runs: `episodic-memory config setup` → enables AO/Arweave
4. Session-end hook continues working automatically

**AO Process Deployment:**
```bash
# Deploy AO process (one-time per user/team)
episodic-memory ao deploy --network mainnet
# Output: Process deployed: p_abc123...def789
#         Save this ID: episodic-memory config set ao-process-id p_abc123...def789

# Or use shared process (team memory)
episodic-memory config set ao-process-id p_shared_team_process
```

**Monitoring and Logging:**

**New Logs:**
```typescript
// src/logging/logger.ts
const logger = {
  info: (msg: string, meta?: object) => console.log(JSON.stringify({ level: 'info', msg, ...meta })),
  warn: (msg: string, meta?: object) => console.warn(JSON.stringify({ level: 'warn', msg, ...meta })),
  error: (msg: string, meta?: object) => console.error(JSON.stringify({ level: 'error', msg, ...meta }))
};

// Usage
logger.info('Uploading to Arweave', { conversationId: 'conv-123', size: '12KB' });
logger.warn('AO process slow', { processId: 'p_abc', latency: 2500 });
logger.error('Arweave upload failed', { conversationId: 'conv-123', error: e.message });
```

**Debugging:**
```bash
# Enable debug logging
export DEBUG=episodic-memory:*
episodic-memory sync

# View AO process state
episodic-memory ao inspect --process-id p_abc123

# Check Arweave transaction status
episodic-memory arweave status --tx TX_abc123
```

**Configuration Management:**

**Config File Location:** `~/.episodic-memory/config.json`

**Schema:**
```typescript
interface Config {
  version: string;
  defaultBackend: 'local' | 'ao' | 'hybrid';

  // AO configuration
  aoProcessId?: string;
  aoNetwork?: 'mainnet' | 'testnet';

  // Arweave configuration
  wallet?: string;  // Path to wallet JSON
  arweaveGateway?: string;  // Default: arweave.net

  // Cache configuration
  cache: {
    enabled: boolean;
    ttl: number;  // milliseconds
    maxSize: number;  // bytes
    strategy: 'recent' | 'frequent' | 'all' | 'none';
  };

  // Project-specific backend
  projects: {
    [projectName: string]: 'local' | 'ao';
  };
}
```

### Risk Assessment and Mitigation

**From document-project - Known Issues:**
- Embedding model locked (can't change without re-index)
- No database version tracking
- Native dependencies (better-sqlite3)
- Single database file

**New Risks from AO/Arweave:**

**Technical Risks:**

1. **AO Process Reliability**
   - Risk: AO process crashes, memory lost
   - Mitigation: SQLite cache provides fallback, can re-index from Arweave
   - Severity: Medium
   - Likelihood: Low (AO designed for reliability)

2. **Arweave Upload Failures**
   - Risk: Network interruption during upload, partial data
   - Mitigation: Atomic uploads (temp file + rename), retry queue
   - Severity: Medium
   - Likelihood: Medium (network dependent)

3. **Wallet Key Loss**
   - Risk: User loses wallet, can't upload new conversations
   - Mitigation: Fallback to local SQLite, clear wallet backup instructions
   - Severity: High (data access loss)
   - Likelihood: Low (user responsibility)

4. **Cost Overruns**
   - Risk: User uploads thousands of conversations, high Arweave costs
   - Mitigation: Cost estimation before upload, configurable project filtering
   - Severity: Medium
   - Likelihood: Low (typical user <1000 conversations)

**Integration Risks:**

1. **MCP Tool Breaking Changes**
   - Risk: Backend swap introduces bugs in search/read tools
   - Mitigation: Comprehensive test coverage for both backends, interface abstraction
   - Severity: High (breaks core functionality)
   - Likelihood: Low (good testing)

2. **Backward Compatibility**
   - Risk: Existing users upgrade, local SQLite breaks
   - Mitigation: Backend selection based on config, defaults to local-only
   - Severity: Critical (existing users affected)
   - Likelihood: Very Low (careful migration)

3. **Performance Degradation**
   - Risk: AO/Arweave queries slower than expected, bad UX
   - Mitigation: SQLite cache layer, async prefetching, performance monitoring
   - Severity: Medium
   - Likelihood: Medium (network dependent)

**Deployment Risks:**

1. **AO Process Deployment Complexity**
   - Risk: Users struggle to deploy AO process, setup abandoned
   - Mitigation: Interactive wizard, pre-deployed shared process option
   - Severity: Medium (blocks feature adoption)
   - Likelihood: Medium (new concept for users)

2. **Dependency Conflicts**
   - Risk: New AO/Arweave SDKs conflict with existing dependencies
   - Mitigation: Lock file discipline, peer dependency management
   - Severity: Medium
   - Likelihood: Low

**Mitigation Strategies:**

**1. Comprehensive Testing:**
- Unit tests for all backend implementations
- Integration tests with real AO testnet and Arweave
- Performance benchmarks (search latency, sync throughput)
- Chaos testing (network failures, AO downtime)

**2. Graceful Degradation:**
- Always fall back to SQLite cache when distributed backend unavailable
- Log errors but continue operation
- Clear error messages with actionable solutions

**3. User Education:**
- Setup wizard with clear explanations
- Documentation with cost estimates
- Warning prompts before expensive operations

**4. Monitoring:**
- Log AO query latencies, Arweave upload success rates
- Alert on repeated failures (suggest fallback to local-only)
- Track cache hit rates (optimize for >80%)

---

## Epic and Story Structure

### Epic Approach

**Epic Structure Decision:** Single comprehensive epic for brownfield enhancement

**Rationale:**
- This is one cohesive feature: decentralized memory via AO/Arweave
- All stories contribute to same goal: cross-device + multi-agent memory
- Stories are sequentially dependent (backend → sync → MCP integration)
- Breaking into multiple epics would create artificial boundaries

**Alternative Considered:** Multiple epics (one per backend, one for integration)
- Rejected: Creates coordination overhead, stories are tightly coupled
- Better: Single epic with clear story sequencing and dependencies

---

## Epic 1: Decentralized Cross-Device & Multi-Agent Memory

**Epic Goal:** Enable episodic-memory to support cross-device agent continuity and multi-agent shared memory through AO/Arweave decentralized backend, while maintaining backward compatibility with local SQLite operation and preserving current autonomous agent behavior.

**Integration Requirements:**
- Must maintain MCP tool interface (Claude's agent behavior unchanged)
- Must support hybrid backend (local SQLite + AO/Arweave)
- Must preserve search performance through intelligent caching
- Must work seamlessly with existing session-end hooks

### Story 1.1: Backend Abstraction Layer

**As a** developer maintaining episodic-memory
**I want** a clean abstraction between search logic and storage backend
**So that** we can swap SQLite for AO/Arweave without changing MCP tools or core search algorithms

**Acceptance Criteria:**

1. `SearchBackend` interface defined with methods: `search()`, `index()`, `sync()`
2. `LocalSQLiteBackend` class implements interface, wrapping current SQLite code
3. Backend factory creates appropriate backend based on configuration
4. Existing test suite passes with LocalSQLiteBackend (zero regressions)
5. MCP server uses backend abstraction (no direct SQLite calls)

**Integration Verification:**
- IV1: All existing unit tests pass (no changes to search logic)
- IV2: MCP tools return identical results with abstracted backend
- IV3: Performance baseline maintained (<10ms local searches)

**Dependencies:** None (foundational story)

**Technical Notes:**
- Use dependency injection pattern (pass backend to functions)
- Avoid global singletons (supports testing with mock backends)
- Keep interface minimal (only methods needed for current + AO/Arweave)

---

### Story 1.2: Configuration Management System

**As a** user setting up AO/Arweave backend
**I want** a simple configuration interface
**So that** I can provide wallet, process ID, and cache preferences without editing JSON files

**Acceptance Criteria:**

1. Config file schema defined with zod validation (wallet, aoProcessId, cache settings)
2. `episodic-memory config set <key> <value>` CLI command works
3. `episodic-memory config get <key>` displays current value
4. `episodic-memory config list` shows all configuration
5. Invalid values rejected with clear error messages (e.g., invalid wallet path)
6. Configuration persisted to `~/.episodic-memory/config.json`
7. Missing config file auto-created with defaults (local-only backend)

**Integration Verification:**
- IV1: Backend factory reads config and selects appropriate backend
- IV2: Configuration changes take effect without restart (reload on next command)
- IV3: Existing users without config file continue using local SQLite

**Dependencies:** Story 1.1 (needs backend factory to consume config)

**Technical Notes:**
- Use zod for schema validation (consistent with mcp-server.ts)
- Store config in user home directory (not project directory)
- Support environment variable overrides (e.g., `EPISODIC_MEMORY_AO_PROCESS_ID`)

---

### Story 1.3: Arweave Upload via Permaweb-MCP

**As a** system syncing conversations to Arweave
**I want** reliable file upload orchestrated through permaweb-mcp tools
**So that** conversations are permanently stored without implementing custom Arweave SDK integration

**Acceptance Criteria:**

1. `uploadToArweave(filePath)` wrapper function calls `mcp__permaweb__uploadToArweave` tool
2. Supports conversation JSONL files and summary text files
3. Extracts transaction ID from tool response (parsing text output)
4. `fetchFromArweave(txId)` uses HTTP gateway fetch (simple, no custom tool needed)
5. Error handling for tool failures (retry logic, fallback to local-only)
6. Progress logging for batch uploads
7. Transaction IDs cached in sync_status table (avoid re-uploads)

**Integration Verification:**
- IV1: Uploaded conversations retrievable from Arweave gateway via HTTPS
- IV2: Transaction tags set correctly by permaweb-mcp tool
- IV3: Upload failures log errors but don't crash sync process (graceful degradation)

**Dependencies:** Story 1.2 (needs SEED_PHRASE in config)

**Technical Notes:**
- **Use permaweb-mcp tools** (NOT custom Arweave SDK) - reduces code by ~80%
- Wallet management handled by permaweb-mcp (SEED_PHRASE env var)
- Fetch via HTTP gateway: `fetch('https://arweave.net/' + txId)` - simple, no tool needed
- Cache transaction IDs in sync_status table (avoid re-uploads)

---

### Story 1.4: AO Process Deployment via Permaweb-MCP

**As a** user enabling AO/Arweave backend
**I want** an AO process deployed through permaweb-mcp tools
**So that** metadata is queryable for fast cross-device searches without custom aoconnect integration

**Acceptance Criteria:**

1. Lua handlers defined for: `IndexConversation`, `SearchConversations`, `GetProjectStats`
2. `IndexConversation` handler stores conversation metadata in Lua table
3. `SearchConversations` handler filters by project, tags, date range
4. Handlers respond within 100ms for typical queries (<1000 conversations)
5. `episodic-memory ao deploy` CLI command:
   - Calls `mcp__permaweb__spawnProcess` to create process
   - Calls `mcp__permaweb__sendAOMessage` with Action: Eval to deploy Lua code
   - Returns Process ID for user configuration
6. Build step concatenates `src/ao/handlers/*.lua` into single deployable bundle
7. Deployed process accessible from multiple devices (shared state)

**Integration Verification:**
- IV1: Spawn process via permaweb-mcp, receive valid Process ID
- IV2: Send Eval message with Lua handlers, process loads code successfully
- IV3: Index 100 test conversations via sendAOMessage, query via readAOProcess returns correct results

**Dependencies:** Story 1.3 (permaweb-mcp configured)

**Technical Notes:**
- **Use permaweb-mcp tools** (NOT custom aoconnect SDK) - reduces deployment code by ~70%
- Process spawning: `mcp__permaweb__spawnProcess` (empty container)
- Code deployment: `mcp__permaweb__sendAOMessage` with `Action: Eval` and Lua code as data
- Store process source in `src/ao/handlers/*.lua`
- Build step: `cat src/ao/handlers/*.lua > dist/ao-process.lua`
- Consider pre-deployed shared process for easier onboarding (users can skip deployment)

---

### Story 1.5: AO Message Wrappers via Permaweb-MCP

**As a** backend querying AO process
**I want** thin wrapper functions around permaweb-mcp tools
**So that** I can index conversations and search metadata without custom aoconnect integration

**Acceptance Criteria:**

1. `indexToAO(processId, conversation, arweaveTx)` wrapper calls `mcp__permaweb__sendAOMessage`
2. `searchAO(processId, query, filters)` wrapper calls `mcp__permaweb__readAOProcess`
3. Functions parse permaweb-mcp tool responses into TypeScript types (ConversationMetadata)
4. Zod schemas validate tool responses before parsing
5. Network errors handled gracefully (retry logic, fallback to cache)
6. Timeout after 5 seconds (prevent hanging on slow AO nodes)

**Integration Verification:**
- IV1: Messages successfully delivered to AO process (verify via permaweb-mcp logs)
- IV2: Search results match expected format (zod validation passes)
- IV3: Timeout/error cases don't crash application (graceful degradation)

**Dependencies:** Story 1.4 (needs deployed AO process to test against)

**Technical Notes:**
- **Use permaweb-mcp tools** (NOT custom aoconnect SDK) - reduces client code by ~90%
- Wrapper pattern: accept high-level params, construct tool calls, parse responses
- Example: `indexToAO()` → construct tags array → call `sendAOMessage` → parse response
- Implement exponential backoff for retries (permaweb-mcp handles basic retry)
- Log all tool calls for debugging (tool name, args, response)

---

### Story 1.6: AOArweaveBackend Implementation

**As a** system with AO/Arweave configured
**I want** a backend implementation that uploads to Arweave and indexes to AO
**So that** conversations are permanently stored and cross-device searchable

**Acceptance Criteria:**

1. `AOArweaveBackend` class implements `SearchBackend` interface
2. `index(exchange)` uploads conversation to Arweave, then indexes metadata to AO
3. `search(query)` queries AO for metadata, fetches content from Arweave
4. `sync()` batch processes multiple conversations with concurrency
5. Results cached in SQLite for fast subsequent searches
6. Graceful fallback to SQLite cache when AO/Arweave unreachable

**Integration Verification:**
- IV1: Indexed conversations searchable immediately after sync
- IV2: Search results identical between local and AO/Arweave backends (same conversation data)
- IV3: Offline searches return cached results (no network errors)

**Dependencies:**
- Story 1.1 (SearchBackend interface)
- Story 1.3 (Arweave upload/download)
- Story 1.5 (AO client)

**Technical Notes:**
- Implement Promise.all for parallel Arweave fetches
- Use sync_status table to track upload progress
- Log upload/index timings for performance monitoring

---

### Story 1.7: SQLite Cache Layer for AO/Arweave

**As a** user with AO/Arweave backend enabled
**I want** frequently accessed conversations cached locally
**So that** searches are fast and work offline

**Acceptance Criteria:**

1. `CacheManager` class handles cache CRUD (insert, get, invalidate)
2. Cache entries expire after configurable TTL (default: 7 days)
3. Cache size limited by total bytes (default: 100MB, LRU eviction)
4. `last_cached` timestamp tracked for each exchange in SQLite
5. Cache hit rate >80% for typical usage (repeated searches)
6. `episodic-memory cache status` shows statistics (size, hit rate, entries)
7. `episodic-memory cache clear` purges all cached data

**Integration Verification:**
- IV1: Second search for same query returns from cache (<10ms)
- IV2: Cache automatically evicts old entries when size limit exceeded
- IV3: Offline search returns cached results (no AO/Arweave calls)

**Dependencies:** Story 1.6 (AOArweaveBackend needs cache)

**Technical Notes:**
- Reuse existing SQLite exchanges table (add cache columns)
- Implement LRU eviction with access_count column
- Track cache hit/miss metrics for monitoring

---

### Story 1.8: Hybrid Sync Workflow

**As a** user with AO/Arweave configured
**I want** `episodic-memory sync` to upload new conversations
**So that** my memory is backed up and accessible cross-device

**Acceptance Criteria:**

1. `sync` command detects backend from config (local or AO/Arweave)
2. Local-only mode: indexes to SQLite (current behavior, unchanged)
3. AO/Arweave mode: parses → uploads to Arweave → indexes to AO → caches to SQLite
4. Batch processing with concurrency (10 conversations in parallel)
5. Progress indicator shows upload status
6. Failures logged but don't stop sync (continue with next conversation)
7. Session-end hook continues working (`hooks/hooks.json` unchanged)

**Integration Verification:**
- IV1: Synced conversations searchable on different device (cross-device test)
- IV2: Local SQLite retains copy (offline access works)
- IV3: Existing local-only users unaffected (no config = local mode)

**Dependencies:**
- Story 1.6 (AOArweaveBackend for sync logic)
- Story 1.7 (Cache for local copy)

**Technical Notes:**
- Use same concurrency pattern as current summarization (processBatch function)
- Store sync status in sync_status table (resumable on failure)
- Emit sync events for monitoring/logging

---

### Story 1.9: MCP Server Backend Integration

**As a** Claude agent searching conversation memory
**I want** MCP tools to work transparently with AO/Arweave backend
**So that** I can access cross-device memory without changing tool usage

**Acceptance Criteria:**

1. MCP `search` tool uses backend abstraction (local or AO/Arweave based on config)
2. MCP `read` tool fetches from Arweave if backend is AO/Arweave, else local SQLite
3. Tool response format unchanged (same markdown output)
4. Search latency acceptable (<1s for cache miss, <10ms for cache hit)
5. Agent autonomously searches memory (no user command changes)
6. Error responses clear and actionable (e.g., "AO unreachable, using cached results")

**Integration Verification:**
- IV1: Agent searches work identically with local and AO/Arweave backends (same queries, same results)
- IV2: Agent references cross-device context (conversation from different device found and used)
- IV3: Offline mode works (cached results returned when AO/Arweave unavailable)

**Dependencies:**
- Story 1.1 (Backend abstraction)
- Story 1.6 (AOArweaveBackend)
- Story 1.8 (Sync workflow for data availability)

**Technical Notes:**
- Modify `src/mcp-server.ts` to use backend factory
- No changes to tool schemas (backward compatible)
- Log backend source in tool responses (for debugging)

---

### Story 1.10: Interactive Setup Wizard

**As a** new user enabling cross-device memory
**I want** a guided setup process
**So that** I can configure AO/Arweave without reading documentation

**Acceptance Criteria:**

1. `episodic-memory config setup` launches interactive wizard
2. Wizard offers: create new AO process OR use existing process ID
3. Wallet options: generate new wallet OR provide existing wallet path
4. Cost estimation shown before creating wallet/process (e.g., "$2 for 1000 conversations")
5. Option to upload existing conversations (with progress bar)
6. Validation at each step (wallet file exists, process ID valid format)
7. Success message with next steps (e.g., "Run: episodic-memory sync")

**Integration Verification:**
- IV1: New users complete setup in <5 minutes
- IV2: Setup creates valid config (backend factory successfully initializes)
- IV3: First sync after setup works (conversations uploaded and searchable)

**Dependencies:**
- Story 1.2 (Config management)
- Story 1.4 (AO deployment)
- Story 1.8 (Sync workflow for initial upload)

**Technical Notes:**
- Use inquirer or prompts for interactive CLI
- Support non-interactive mode (--wallet, --process-id flags)
- Provide option to skip initial upload (upload later)

---

### Story 1.11: Cross-Device Sync Verification

**As a** user working across multiple devices
**I want** my agent to remember context from other devices
**So that** I have seamless continuity regardless of where I work

**Acceptance Criteria:**

1. **Device A:** Index conversation about authentication via `episodic-memory sync`
2. **Device B:** Run `episodic-memory sync` (downloads metadata from AO)
3. **Device B:** Agent searches for "authentication" and finds Device A's conversation
4. **Device B:** Agent correctly answers question using Device A's context
5. Cross-device latency acceptable (first search <2s, subsequent <10ms via cache)
6. No manual intervention required (automatic via hooks and MCP tools)

**Integration Verification:**
- IV1: Multi-device workflow tested end-to-end (two actual machines or VMs)
- IV2: Agent on Device B references specific details from Device A conversation
- IV3: SQLite cache on Device B populated after first search (offline use works)

**Dependencies:**
- Story 1.8 (Sync workflow)
- Story 1.9 (MCP integration)

**Technical Notes:**
- This is an integration test story (validates entire flow)
- Test with real devices, not mocks (verify network behavior)
- Document user experience for testimonial/demo

---

### Story 1.12: Multi-Agent Shared Memory

**As a** team using multiple specialized agents (dev, PM, QA)
**I want** agents to share a knowledge base
**So that** insights from one agent are accessible to others

**Acceptance Criteria:**

1. Multiple users configure same AO process ID (shared process)
2. Dev agent indexes conversation about bug fix
3. PM agent queries "recent bug fixes" and finds dev agent's conversation
4. QA agent queries same process and accesses both dev and PM contexts
5. Access control prevents unauthorized reads (future enhancement noted)
6. Concurrent queries from multiple agents handled correctly (no race conditions)

**Integration Verification:**
- IV1: Three different agent instances (dev, PM, QA) query same AO process
- IV2: Knowledge created by one agent visible to others within seconds
- IV3: AO process state remains consistent under concurrent writes

**Dependencies:**
- Story 1.4 (AO process with shared state)
- Story 1.5 (AO client for concurrent access)
- Story 1.9 (MCP integration for agent queries)

**Technical Notes:**
- Test with 3+ concurrent clients
- Verify AO process handles message ordering correctly
- Document shared process setup in README

---

### Story 1.13: Performance Optimization & Monitoring

**As a** system operator
**I want** to monitor search performance and cache efficiency
**So that** I can optimize for speed and identify bottlenecks

**Acceptance Criteria:**

1. Performance metrics logged: search latency, AO query time, Arweave fetch time, cache hit rate
2. `episodic-memory stats` shows performance breakdown
3. Cache hit rate >80% for typical usage patterns
4. AO queries <200ms (95th percentile)
5. Arweave fetches <1s (95th percentile)
6. Prefetch optimization: frequently searched conversations cached proactively
7. Background cache warming on startup (recent 100 conversations)

**Integration Verification:**
- IV1: Performance benchmarks run against real AO/Arweave (not mocks)
- IV2: Cache warming reduces first-search latency by >50%
- IV3: Metrics logged to file for analysis (episodic-memory-metrics.json)

**Dependencies:** Story 1.9 (MCP integration for real-world usage patterns)

**Technical Notes:**
- Use high-resolution timers for accurate latency measurement
- Implement percentile tracking (not just averages)
- Consider telemetry opt-in for aggregate performance data

---

### Story 1.14: Error Handling & Graceful Degradation

**As a** user experiencing network issues or AO downtime
**I want** episodic-memory to continue functioning
**So that** my agent can still search cached conversations

**Acceptance Criteria:**

1. AO unreachable: Fall back to SQLite cache, log warning
2. Arweave gateway slow: Timeout after 5s, retry once, then use cache
3. Wallet errors: Clear error message with solution (e.g., "Wallet file not found at X")
4. Invalid config: Validation errors before sync starts (not mid-process)
5. Partial sync failures: Log failed conversations, continue with next
6. Network timeouts: Exponential backoff, max 3 retries
7. All errors logged to `~/.episodic-memory/error.log`

**Integration Verification:**
- IV1: Chaos testing (disconnect network mid-sync, verify graceful handling)
- IV2: Agent continues working offline (cached results returned)
- IV3: Error messages actionable (user knows how to fix)

**Dependencies:** All previous stories (tests error paths across system)

**Technical Notes:**
- Create error taxonomy (NetworkError, ConfigError, etc.)
- Test all error scenarios (unit + integration tests)
- Provide runbook for common errors in docs

---

### Story 1.15: Documentation & Migration Guide

**As a** existing episodic-memory user
**I want** clear documentation for AO/Arweave setup
**So that** I can opt-in to cross-device memory confidently

**Acceptance Criteria:**

1. README updated with AO/Arweave feature description
2. Setup guide with screenshots (or CLI output examples)
3. Migration guide for existing users (how to upload local conversations)
4. Cost estimation table (conversations per dollar at current Arweave prices)
5. Troubleshooting section for common errors
6. Architecture diagram showing hybrid backend flow
7. API documentation for new backend abstraction (JSDoc comments)

**Integration Verification:**
- IV1: New user follows setup guide, successfully enables AO/Arweave
- IV2: Existing user follows migration guide, uploads local history
- IV3: Troubleshooting guide resolves top 3 user errors

**Dependencies:** All previous stories (documents completed features)

**Technical Notes:**
- **Create** `docs/architecture.md` with target architecture (v2.0 with AO/Arweave + Permamind)
- **Keep** `docs/brownfield-architecture.md` as current state baseline (v1.0.9)
- Document MCP orchestration pattern (how permaweb-mcp and permamind-mcp are used)
- Include architecture diagrams: hybrid backend, hierarchical search, cross-device sync
- Add examples to README (before/after for cross-device scenario)
- Create FAQ section for common questions

---

### Story 1.16: Permamind Integration - Hierarchical Memory Search

**As a** agent searching for knowledge
**I want** to search local project memory first, then fall back to global skill registry
**So that** I find project-specific context when available, or learn from other projects when not

**Acceptance Criteria:**

1. MCP `search` tool implements hierarchical search:
   - **Layer 1:** Search local episodic-memory (project-specific conversations)
   - **Layer 2:** If Layer 1 has <3 results, search permamind registry (global skills)
   - **Layer 3:** Clearly indicate source ("Project Memory" vs "Global Skills")
2. Search results show both local conversations AND global skills (when relevant)
3. Agent autonomously decides to search permamind when local memory insufficient
4. Results formatted with source tags: `[Project Memory]` or `[Skill: skill-name]`

**Integration Verification:**
- IV1: Search with local results returns ONLY episodic-memory (prefers project context)
- IV2: Search with NO local results automatically searches permamind registry
- IV3: Search with few local results shows BOTH local + global (enriched context)
- IV4: Agent correctly uses permamind skills when local memory doesn't exist

**Dependencies:** Story 1.9 (MCP integration complete)

**Technical Notes:**
- **CORE MVP FEATURE** - enables cross-project learning from day one
- Implements hierarchical search pattern (local first, global fallback)
- Use `mcp__permamind__search_skills` for Layer 2 search
- Result format distinguishes sources clearly for agent reasoning
- Synergy: episodic-memory = project memory, permamind = global memory

---

### Story 1.17: Conversation-to-Skill Publishing

**As a** user who solved a complex problem
**I want** to publish valuable conversations as skills to permamind registry
**So that** other agents/projects can learn from my solutions

**Acceptance Criteria:**

1. `episodic-memory publish-skill <conversation-id>` CLI command extracts insights
2. Interactive wizard prompts for:
   - Skill title (suggested from conversation topic)
   - Tags (suggested from tools used, languages detected)
   - Description (auto-generated from summary)
3. Skill structure generated:
   - `SKILL.md` with conversation summary + key decisions + code snippets
   - Tagged with: topic, project, tools-used, language, framework
4. Uses `mcp__permamind__publish_skill` to publish to Arweave registry
5. Published skill immediately searchable (verify with permamind search)
6. Skill references original conversation (link back to episodic-memory)

**Integration Verification:**
- IV1: Published skill found via `mcp__permamind__search_skills` within 30 seconds
- IV2: Skill content accurately represents conversation insights
- IV3: Agent in different project finds and uses published skill successfully

**Dependencies:** Story 1.16 (permamind search integration)

**Technical Notes:**
- Use Claude SDK to extract "key insights" from conversations (summarization)
- Skill structure: SKILL.md with metadata + conversation excerpt + code examples
- Auto-suggest based on heuristics: >5 tool calls, code generation, user said "remember this"
- Store original conversation ID in skill metadata (bidirectional linking)

---

### Story 1.18: Auto-Publish Recommendations

**As a** user completing a valuable conversation
**I want** episodic-memory to suggest publishing as skill automatically
**So that** I don't miss opportunities to share knowledge

**Acceptance Criteria:**

1. After conversation ends (session-end hook), analyze for skill-worthiness:
   - **High tool usage:** >10 tool calls (complex solution)
   - **Code generation:** >100 lines of code written
   - **User signal:** User said "remember this" or "save this pattern"
   - **Problem-solution:** User asked question → Agent solved → User confirmed success
2. If skill-worthy, prompt user: "Publish this conversation as skill? [y/N]"
3. If yes, launch `publish-skill` wizard (Story 1.17)
4. If no, track decision (don't re-prompt for same conversation)
5. Option to disable auto-suggestions: `episodic-memory config set auto-suggest-publish false`

**Integration Verification:**
- IV1: Complex conversation (>10 tools) triggers suggestion
- IV2: User accepts suggestion → skill published successfully
- IV3: User declines suggestion → not prompted again for that conversation
- IV4: Disabled auto-suggest → no prompts appear

**Dependencies:** Story 1.17 (publish-skill command exists)

**Technical Notes:**
- **POLISH FEATURE** - can be last, after core publishing works
- Heuristics tunable via config (e.g., min tool calls threshold)
- Non-blocking: suggestion shown AFTER sync complete (don't slow down session-end)
- Track suggestions in SQLite: `suggested_skills` table (conversation_id, suggested_at, user_action)

---

## Story Dependency Graph

```
1.1 Backend Abstraction (foundational)
  ↓
1.2 Configuration Management
  ↓
  ├─→ 1.3 Arweave Upload via Permaweb-MCP
  │     ↓
  └─→ 1.4 AO Process Deployment via Permaweb-MCP
        ↓
        1.5 AO Message Wrappers via Permaweb-MCP
        ↓
        1.6 AOArweaveBackend Implementation (orchestrates permaweb-mcp tools)
        ↓
        1.7 SQLite Cache Layer
        ↓
        1.8 Hybrid Sync Workflow
        ↓
        1.9 MCP Server Integration
        ↓
        ├─→ 1.10 Setup Wizard
        ├─→ 1.11 Cross-Device Verification (integration test)
        ├─→ 1.12 Multi-Agent Shared Memory (integration test)
        ├─→ 1.13 Performance Optimization
        ├─→ 1.14 Error Handling
        └─→ 1.16 Permamind Hierarchical Search (CORE MVP)
              ↓
              1.17 Conversation-to-Skill Publishing
              ↓
              1.18 Auto-Publish Recommendations (POLISH)
              ↓
              1.15 Documentation
```

**Critical Path:** 1.1 → 1.2 → 1.3/1.4 → 1.5 → 1.6 → 1.8 → 1.9 → 1.16 → 1.11 (cross-device + hierarchical search)

**Parallel Tracks:**
- Track A: 1.3 Arweave Upload (can develop in parallel with 1.4)
- Track B: 1.7 Cache Layer (can develop after 1.6 interface defined)
- Track C: 1.10 Setup Wizard (UX polish, can be last)
- Track D: 1.13 Performance Optimization (after core features working)
- Track E: 1.17 → 1.18 Publishing features (after hierarchical search works)

**MVP Definition (Stories 1.1-1.17):**
- Cross-device memory via AO/Arweave ✅
- Multi-agent shared memory ✅
- Hierarchical search (local → global) ✅
- Conversation-to-skill publishing ✅
- Auto-publish suggestions = nice-to-have (Story 1.18)

**Key Simplifications via MCP Orchestration:**
- Stories 1.3, 1.4, 1.5: Now thin wrappers (~50-100 lines each vs 200-500 lines)
- Stories 1.16, 1.17: Leverage permamind-mcp tools (no custom registry implementation)
- No custom Arweave SDK integration needed
- No custom aoconnect client needed
- Wallet management handled by permaweb-mcp (SEED_PHRASE env var)

---

## Risk Mitigation per Story

**High-Risk Stories:**

**Story 1.6 (AOArweaveBackend):** Most complex integration
- **Mitigation:** Comprehensive mocks for testing, test against real AO testnet early
- **Rollback:** Backend abstraction allows falling back to local-only

**Story 1.9 (MCP Integration):** Affects all agent workflows
- **Mitigation:** Extensive testing with both backends, gradual rollout
- **Rollback:** Config flag to force local-only mode

**Story 1.11 (Cross-Device Verification):** End-to-end validation
- **Mitigation:** Test on real devices (not just localhost), document setup precisely
- **Rollback:** N/A (integration test, not production code)

---

## Success Metrics

**Functional Success:**
- ✅ Cross-device: Agent on Device B answers using Device A's context
- ✅ Multi-agent: PM agent references dev agent's conversation
- ✅ Backward compat: Existing users upgrade without errors (local-only continues working)

**Performance Success:**
- ✅ Cache hit rate: >80%
- ✅ Cached search: <10ms (baseline)
- ✅ AO query: <200ms (95th percentile)
- ✅ Arweave fetch: <1s (95th percentile)

**User Success:**
- ✅ Setup time: <5 minutes (from config setup to first sync)
- ✅ Error rate: <5% of syncs fail
- ✅ User satisfaction: Positive feedback on cross-device continuity

---

**🎯 Epic Complete When:**
- **MVP Stories 1.1-1.17 DONE:**
  - ✅ Cross-device memory (Stories 1.1-1.12)
  - ✅ Multi-agent collaboration (Story 1.12)
  - ✅ Hierarchical search - local → global (Story 1.16)
  - ✅ Skill publishing (Story 1.17)
- Story 1.18 (Auto-publish) = nice-to-have polish
- Integration tests passing (cross-device, multi-agent, hierarchical search)
- Documentation published
- Performance benchmarks meet targets
- Zero regressions in existing functionality
- Plugin configuration updated with permaweb-mcp and permamind-mcp servers
- Demo: Agent finds answer in global skills when local project has no context

---

**Document Version:** 2.0
**Last Updated:** 2025-11-07
**Status:** Ready for Development
**Approver:** Product Management

**Revision History:**
- v1.0 (2025-11-07): Initial PRD with custom AO/Arweave integration
- v1.1 (2025-11-07): Revised to use MCP orchestration pattern (permaweb-mcp, permamind-mcp)
- v2.0 (2025-11-07): **Permamind integration promoted to CORE MVP** - Added Stories 1.16-1.18 for hierarchical search and skill publishing

---

_🤖 Generated with [Claude Code](https://claude.com/claude-code)_

_Co-Authored-By: Claude <noreply@anthropic.com>_
