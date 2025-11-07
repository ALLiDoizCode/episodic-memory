# Architecture Document Guide

## Purpose

This guide clarifies the architecture documentation strategy for the Episodic Memory v2.0 enhancement.

---

## Two-Document Strategy

### 📄 `brownfield-architecture.md` (Current State - DO NOT MODIFY)

**Status:** AS-BUILT (v1.0.9 - Production)
**Last Updated:** 2025-11-07
**Purpose:** Baseline snapshot of CURRENT implementation for brownfield enhancement reference

**Contains:**
- Current local SQLite implementation
- Existing MCP server (episodic-memory only)
- Current module organization
- Technical debt and known issues
- Actual tech stack (v1.0.9)

**Do NOT update this file** - it serves as the "before" snapshot for comparing changes.

---

### 📄 `architecture.md` (Target State - CREATE THIS)

**Status:** PLANNED (PRD v2.0)
**Last Updated:** TBD (Architect creates)
**Purpose:** TARGET architecture with AO/Arweave + Permamind integration

**Should Contain:**

#### 1. System Overview
- High-level architecture diagram showing:
  - Hybrid backend (SQLite cache + AO metadata + Arweave content)
  - MCP orchestration pattern (episodic-memory → permaweb-mcp → AO/Arweave)
  - Permamind integration (hierarchical search)
  - Plugin configuration (three MCP servers)

#### 2. Component Architecture

**Backend Abstraction Layer:**
- `SearchBackend` interface
- `LocalSQLiteBackend` implementation
- `AOArweaveBackend` implementation
- Backend factory pattern

**MCP Server Orchestration:**
- How episodic-memory MCP server calls permaweb-mcp tools
- How episodic-memory MCP server calls permamind-mcp tools
- Tool orchestration pattern (ClaudeCodeSDK.callTool)

**AO Integration (via permaweb-mcp):**
- Process spawning (`mcp__permaweb__spawnProcess`)
- Message sending (`mcp__permaweb__sendAOMessage`)
- Process querying (`mcp__permaweb__readAOProcess`)
- Lua handler architecture (IndexConversation, SearchConversations)

**Arweave Integration (via permaweb-mcp):**
- Upload strategy (`mcp__permaweb__uploadToArweave`)
- Download strategy (HTTP gateway fetch)
- Transaction tagging
- Cost optimization

**Permamind Integration (via permamind-mcp):**
- Hierarchical search pattern (local → global fallback)
- Skill publishing workflow
- Skill format (SKILL.md structure)

**SQLite Cache Layer:**
- Cache invalidation strategy (TTL-based)
- Cache schema updates (arweave_tx, last_cached columns)
- LRU eviction policy

#### 3. Data Flow Diagrams

**Conversation Indexing Flow:**
```
Parse JSONL
  → Upload to Arweave (via permaweb-mcp)
  → Index to AO (via permaweb-mcp)
  → Cache to SQLite
```

**Cross-Device Sync Flow:**
```
Device A: Conversation → Arweave + AO
  ↓ (shared AO process)
Device B: Query AO → Fetch Arweave → Cache SQLite → Search
```

**Hierarchical Search Flow:**
```
Agent search query
  → Search local episodic-memory
  → If insufficient (<3 results)
    → Search permamind registry (via permamind-mcp)
  → Merge results with source tags
  → Return to agent
```

**Skill Publishing Flow:**
```
User: publish-skill <conversation-id>
  → Extract insights (Claude SDK summarization)
  → Generate SKILL.md
  → Publish via mcp__permamind__publish_skill
  → Verify searchable in registry
```

#### 4. Database Design

**SQLite Schema Updates:**
```sql
-- Add cache metadata to exchanges table
ALTER TABLE exchanges ADD COLUMN arweave_tx TEXT;
ALTER TABLE exchanges ADD COLUMN last_cached INTEGER;
ALTER TABLE exchanges ADD COLUMN cache_ttl INTEGER;

-- Add sync status tracking
CREATE TABLE sync_status (
  conversation_id TEXT PRIMARY KEY,
  synced_to_ao BOOLEAN,
  synced_to_arweave BOOLEAN,
  arweave_tx TEXT,
  ao_indexed_at INTEGER,
  last_sync_attempt INTEGER,
  sync_error TEXT
);

-- Add skill suggestion tracking
CREATE TABLE suggested_skills (
  conversation_id TEXT PRIMARY KEY,
  suggested_at INTEGER,
  user_action TEXT, -- 'accepted', 'declined', 'deferred'
  skill_id TEXT
);
```

**AO Process Lua Table Structure:**
```lua
Conversations = {
  ["conv-id-1"] = {
    id = "conv-id-1",
    project = "my-app",
    timestamp = "2025-11-07T14:00:00Z",
    arweave_conversation = "TX_ABC123",
    arweave_summary = "TX_DEF456",
    tool_names = {"Read", "Edit", "Bash"},
    tags = {"bug-fix", "authentication"},
    snippet = "First 200 chars...",
    session_id = "session-abc",
    git_branch = "main"
  }
}

Projects = {
  ["my-app"] = {
    conversation_count = 1523,
    last_updated = "2025-11-07T14:00:00Z",
    conversation_ids = {"conv-id-1", "conv-id-2", ...}
  }
}

Tags = {
  ["bug-fix"] = {"conv-id-1", "conv-id-12"},
  ["authentication"] = {"conv-id-1", "conv-id-45"}
}
```

**Arweave Data Model:**
- JSONL files tagged: `conversation-id`, `project`, `timestamp`, `session-id`
- Summary files tagged: `conversation-id`, `type: summary`
- Transaction IDs stored in AO process + SQLite

#### 5. Integration Patterns

**MCP Tool Orchestration Pattern:**
```typescript
// Pattern: episodic-memory MCP server calls other MCP tools
import { ClaudeCodeSDK } from '@anthropic-ai/claude-agent-sdk';

const claude = new ClaudeCodeSDK();

// Instead of custom SDK:
const result = await claude.callTool('mcp__permaweb__uploadToArweave', {
  filePath: conversationPath,
  paymentMethod: 'tokens'
});

const txId = extractTxId(result.content[0].text);
```

**Error Handling Strategy:**
- Always fall back to SQLite cache when AO/Arweave unreachable
- Log errors but continue operation
- Retry with exponential backoff (3 attempts)
- Clear error messages with actionable solutions

**Offline Mode:**
- Search works against SQLite cache
- Sync queued until online
- No network errors surface to user

#### 6. Plugin Architecture

**Plugin Configuration (.claude-plugin/plugin.json):**
```json
{
  "mcpServers": {
    "episodic-memory": { /* own MCP server */ },
    "permaweb": { /* AO/Arweave operations */ },
    "permamind": { /* skill registry */ }
  }
}
```

**MCP Server Coordination:**
- episodic-memory MCP provides `search` and `read` tools to Claude
- episodic-memory internally calls permaweb and permamind tools
- All three servers share SEED_PHRASE env var for wallet

#### 7. Security & Privacy

**Wallet Management:**
- SEED_PHRASE environment variable (12-word mnemonic)
- Handled by permaweb-mcp and permamind-mcp
- Never logged or stored in code

**Access Control (Future):**
- AO process can check message sender (wallet address)
- Implement read/write permissions per project
- Private vs public conversation separation

**Data Privacy:**
- Local SQLite cache remains private
- Arweave uploads are PUBLIC by default
- Optional: Encrypt conversations before upload (future enhancement)

#### 8. Performance Considerations

**Search Performance:**
- Local SQLite: <10ms (baseline maintained)
- SQLite cache hit: <10ms (no degradation)
- AO query: <200ms target (95th percentile)
- Arweave fetch: <1s target (95th percentile)
- Permamind search: <500ms target

**Cache Strategy:**
- Default TTL: 7 days
- LRU eviction when size >100MB
- Prefetch recent 100 conversations on startup
- Cache hit rate target: >80%

**Sync Performance:**
- Arweave upload: <5s per conversation
- Batch concurrency: 10 parallel uploads
- Background sync (non-blocking)

#### 9. Deployment Architecture

**Development:**
```
Local Machine
├── SQLite: ~/.claude/conversation-search/conversations.db
├── Archive: ~/.claude/conversation-search/archive/
├── MCP Servers: Started by Claude Code plugin
│   ├── episodic-memory (local)
│   ├── permaweb-mcp (npx)
│   └── permamind-mcp (npx)
└── AO Process: Testnet or shared process
```

**Production:**
```
User Machine (Device A, B, C...)
├── SQLite Cache (local, fast)
├── MCP Servers (plugin-managed)
└── Network:
    ├── AO Process (shared, metadata index)
    ├── Arweave (permanent content storage)
    └── Permamind Registry (global skills)
```

---

## Key Architectural Decisions

### 1. **MCP Orchestration over Custom SDKs**
- **Decision:** Use permaweb-mcp and permamind-mcp tools instead of integrating aoconnect and arweave SDKs directly
- **Rationale:** Reduces code by ~70%, leverages tested implementations, better separation of concerns
- **Tradeoff:** Dependency on external MCP servers (mitigation: fallback to SQLite)

### 2. **Hybrid Backend (SQLite + AO/Arweave)**
- **Decision:** SQLite as cache, AO/Arweave as source of truth when configured
- **Rationale:** Maintains local speed, enables cross-device, graceful degradation
- **Tradeoff:** Dual-backend complexity (mitigation: clean abstraction layer)

### 3. **Hierarchical Search (Local → Global)**
- **Decision:** Search local episodic-memory first, permamind second
- **Rationale:** Prefer project-specific context, fall back to global knowledge
- **Tradeoff:** Slower when no local results (mitigation: automatic fallback, user unaware)

### 4. **Opt-In Enhancement**
- **Decision:** Local SQLite works without configuration, AO/Arweave requires setup
- **Rationale:** Backward compatibility, zero-friction onboarding, progressive enhancement
- **Tradeoff:** Two code paths to maintain (mitigation: backend abstraction isolates)

---

## Architect Deliverables

When creating `docs/architecture.md`, include:

✅ **Diagrams:**
- System architecture (components + data flow)
- Cross-device sync sequence
- Hierarchical search flow
- MCP orchestration pattern

✅ **Component Specifications:**
- Backend abstraction interfaces
- AO Lua handler APIs
- SQLite schema changes
- MCP tool usage patterns

✅ **API Documentation:**
- Public API (from index.ts)
- Internal backend APIs
- AO process message formats
- Permamind skill format

✅ **Deployment Guide:**
- Development setup
- Production deployment
- AO process deployment
- Plugin configuration

✅ **Cross-References:**
- Link to `prd.md` for requirements
- Link to `brownfield-architecture.md` for current state
- Link to `SCHEMA.md` for database details

---

## Questions for Architect?

Review:
- `docs/prd.md` (requirements and stories)
- `docs/brownfield-architecture.md` (current state)
- This guide (what to design)

Then create `docs/architecture.md` with target architecture for v2.0!
