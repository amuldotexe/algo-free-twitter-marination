# RD09: Parseltongue Query Cookbook

## A Practical Guide to Code Graph Analysis

This document captures all queries executed during real-world codebase analysis, serving as a reference for Parseltongue users.

---

## Table of Contents

1. [Setup & Installation](#1-setup--installation)
2. [Basic Usage](#2-basic-usage)
3. [Core HTTP Endpoints](#3-core-http-endpoints)
4. [Real-World Query Examples](#4-real-world-query-examples)
5. [Entity Key Format](#5-entity-key-format)
6. [Language Support](#6-language-support)
7. [Common Query Patterns](#7-common-query-patterns)

---

## 1. Setup & Installation

### 1.1 Download Parseltongue

```bash
# Download the binary (one command)
curl -L https://github.com/that-in-rust/parseltongue-dependency-graph-generator/releases/download/v1.2.0/parseltongue -o parseltongue && chmod +x parseltongue

# Verify installation
./parseltongue --version
# Expected: parseltongue 1.2.0

# Optional: Add to PATH for global access
sudo mv parseltongue /usr/local/bin/
```

**Releases**: https://github.com/that-in-rust/parseltongue-dependency-graph-generator/releases

**Size**: ~50MB single binary, zero runtime dependencies

### 1.2 Index Your Codebase

```bash
parseltongue pt01-folder-to-cozodb-streamer ./my-project --db "rocksdb:mycode.db"
```

**Expected Output**:
```
Running Tool 1: folder-to-cozodb-streamer
  Database: rocksdb:mycode.db

Streaming Summary:
Total files found: 108
Files processed: 92
Entities created: 216 (CODE only)
  └─ CODE entities: 216
  └─ TEST entities: 982 (excluded for optimal LLM context)

✓ Indexing completed
```

**Key Points**:
- Always use `rocksdb:` prefix for persistent databases
- Test entities are automatically excluded for optimal LLM context
- Supports 12 languages: Rust, Python, JavaScript, TypeScript, Go, Java, C, C++, Ruby, PHP, C#, Swift

### 1.3 Start the HTTP Server

```bash
parseltongue pt08-http-code-query-server --db "rocksdb:mycode.db"
```

**Expected Output**:
```
Parseltongue HTTP Server
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HTTP Server running at: http://localhost:7777

┌─────────────────────────────────────────────────────────────────┐
│  Add to your LLM agent: PARSELTONGUE_URL=http://localhost:7777  │
└─────────────────────────────────────────────────────────────────┘

Quick test:
  curl http://localhost:7777/server-health-check-status
```

**Default**: Port 7777 (can be changed with `--port <PORT>`)

---

## 2. Basic Usage

### 2.1 Health Check

```bash
curl http://localhost:7777/server-health-check-status
```

### 2.2 Codebase Overview

```bash
curl http://localhost:7777/codebase-statistics-overview-summary
```

### 2.3 Quick Test Queries

```bash
# Search for functions
curl "http://localhost:7777/code-entities-search-fuzzy?q=authenticate"

# What calls this function?
curl "http://localhost:7777/reverse-callers-query-graph?entity=rust:fn:process:src_lib_rs:50-100"

# What breaks if I change this?
curl "http://localhost:7777/blast-radius-impact-analysis?entity=rust:fn:new:src_storage_rs:10-30&hops=3"

# Get optimal context for LLM
curl "http://localhost:7777/smart-context-token-budget?focus=rust:fn:main:src_main_rs:1-50&tokens=4000"
```

---

## 3. Core HTTP Endpoints (15 Total)

### 3.1 Core Endpoints

| Endpoint | Description | Token Cost |
|----------|-------------|------------|
| `GET /server-health-check-status` | Server health check | ~35 |
| `GET /codebase-statistics-overview-summary` | Entity/edge counts, languages | ~100 |
| `GET /api-reference-documentation-help` | Full API documentation | ~500 |

### 3.2 Entity Endpoints

| Endpoint | Description | Token Cost |
|----------|-------------|------------|
| `GET /code-entities-list-all` | All entities | ~2K |
| `GET /code-entities-list-all?entity_type=function` | Filter by type | ~1K |
| `GET /code-entity-detail-view?key=X` | Single entity details | ~200 |
| `GET /code-entities-search-fuzzy?q=pattern` | Fuzzy search by name | ~500 |

### 3.3 Graph Query Endpoints

| Endpoint | Description | Token Cost |
|----------|-------------|------------|
| `GET /dependency-edges-list-all` | All dependency edges | ~3K |
| `GET /reverse-callers-query-graph?entity=X` | Who calls X? | ~500 |
| `GET /forward-callees-query-graph?entity=X` | What does X call? | ~500 |
| `GET /blast-radius-impact-analysis?entity=X&hops=N` | What breaks if X changes? | ~2K |

### 3.4 Analysis Endpoints

| Endpoint | Description | Token Cost |
|----------|-------------|------------|
| `GET /circular-dependency-detection-scan` | Find circular dependencies | ~1K |
| `GET /complexity-hotspots-ranking-view?top=N` | Complexity ranking | ~500 |
| `GET /semantic-cluster-grouping-list` | Semantic module groups | ~1K |

### 3.5 Context Optimization

| Endpoint | Description | Token Cost |
|----------|-------------|------------|
| `GET /smart-context-token-budget?focus=X&tokens=N` | Context selection within token budget | ~4K |
| `GET /temporal-coupling-hidden-deps?entity=X` | Temporal dependency detection | ~200 |

---

## 4. Real-World Query Examples

### 4.1 Large-Scale Codebase Analysis (Apache Iggy)

**Codebase Stats**: 1,845 files, 10,362 entities, 41,255 edges, zero circular dependencies

```bash
# Get comprehensive overview
curl http://localhost:7777/codebase-statistics-overview-summary | jq '.data'
```

**Real Response**:
```json
{
  "code_entities_total_count": 10362,
  "test_entities_total_count": 3758,
  "dependency_edges_total_count": 41255,
  "languages_detected_list": ["rust", "csharp", "go", "cpp", "java"],
  "database_file_path": "rocksdb:iggy-analysis.db"
}
```

### 4.2 Entity Discovery by Domain

```bash
# Find all streaming-related entities
curl "http://localhost:7777/code-entities-search-fuzzy?q=streaming" | jq '.data.total_count'
```

**Real Counts from Apache Iggy**:
- `streaming`: **751 entities**
- `partition`: **534 entities**
- `consumer_group`: **352 entities**
- `shard`: **347 entities**
- `segment`: **319 entities**
- `message`: **1,339 entities**

### 4.3 Impact Analysis on Core Traits

```bash
# Analyze blast radius of BytesSerializable trait
curl "http://localhost:7777/blast-radius-impact-analysis?entity=rust:trait:BytesSerializable:core_common_src_traits_bytes_serializable_rs:23-41&hops=2" | jq '.data'
```

**Real Impact Analysis**:
```json
{
  "source_entity": "rust:trait:BytesSerializable:...",
  "hops_requested": 2,
  "total_affected": 117,
  "by_hop": [
    {"hop": 1, "count": 69, "entities": ["Commands", "Types", "State"]},
    {"hop": 2, "count": 48, "entities": ["HTTP Handlers", "Binary Handlers"]}
  ]
}
```

**Key Finding**: Changing `BytesSerializable` affects **117 entities** across 2 hops - critical protocol impact.

### 4.4 Understanding a New Codebase

```bash
# Get codebase overview
curl http://localhost:7777/codebase-statistics-overview-summary | jq '.data'
```

**Real Response**:
```json
{
  "code_entities_total_count": 217,
  "test_entities_total_count": 0,
  "dependency_edges_total_count": 3027,
  "languages_detected_list": ["rust"],
  "database_file_path": "rocksdb:parseltongue.db"
}
```

```bash
# Find complexity hotspots (most called functions)
curl "http://localhost:7777/complexity-hotspots-ranking-view?top=5" | jq '.data.hotspots'
```

**Real Response**:
```json
[
  {"rank": 1, "entity_key": "rust:fn:new:unknown:0-0", "inbound_count": 215, "total_coupling": 215},
  {"rank": 2, "entity_key": "rust:fn:unwrap:unknown:0-0", "inbound_count": 163, "total_coupling": 163},
  {"rank": 3, "entity_key": "rust:fn:to_string:unknown:0-0", "inbound_count": 139, "total_coupling": 139},
  {"rank": 4, "entity_key": "rust:fn:Ok:unknown:0-0", "inbound_count": 101, "total_coupling": 101},
  {"rank": 5, "entity_key": "rust:fn:Some:unknown:0-0", "inbound_count": 62, "total_coupling": 62}
]
```

> Note: `unknown:0-0` indicates stdlib/external calls (HashMap::new, unwrap, etc.)

```bash
# Check for circular dependencies
curl http://localhost:7777/circular-dependency-detection-scan | jq '.data'
```

**Real Response**:
```json
{"has_cycles": false, "cycle_count": 0, "cycles": []}
```

### 4.2 Impact Analysis Before Refactoring

```bash
# Find who calls CozoDbStorage::new() (reverse dependencies)
curl "http://localhost:7777/reverse-callers-query-graph?entity=rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54" | jq '.data.total_count'
```

**Real Response**: `215` callers!

```bash
# Get full blast radius (2-hop transitive impact)
curl "http://localhost:7777/blast-radius-impact-analysis?entity=rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54&hops=2" | jq '.data'
```

**Real Response**:
```json
{
  "source_entity": "rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54",
  "hops_requested": 2,
  "total_affected": 302,
  "by_hop": [
    {"hop": 1, "count": 214, "entities": ["rust:fn:build_cli:...", "rust:fn:start_http_server_blocking_loop:...", "..."]},
    {"hop": 2, "count": 88, "entities": ["rust:fn:main:...", "rust:fn:handle_blast_radius_impact_analysis:...", "..."]}
  ]
}
```

### 4.3 Finding and Exploring Code

```bash
# Search for storage-related entities
curl "http://localhost:7777/code-entities-search-fuzzy?q=storage" | jq '.data.total_count'
```

**Real Response**: `36` matching entities (struct, impl, methods)

```bash
# Get full source code of CozoDbStorage::new()
curl "http://localhost:7777/code-entity-detail-view?key=rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54" | jq '.data.code'
```

**Real Response**:
```json
{
  "key": "rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54",
  "file_path": "./crates/parseltongue-core/src/storage/cozo_client.rs",
  "entity_type": "method",
  "language": "rust",
  "code": "    pub async fn new(engine_spec: &str) -> Result<Self> {\n        let (engine, path) = if engine_spec.contains(':') {\n            let parts: Vec<&str> = engine_spec.splitn(2, ':').collect();\n            (parts[0], parts[1])\n        } else {\n            (engine_spec, \"\")\n        };\n        let db = DbInstance::new(engine, path, Default::default())\n            .map_err(|e| ParseltongError::DatabaseError {...})?;\n        Ok(Self { db })\n    }"
}
```

### 4.4 Architecture Health Assessment

```bash
# Check for circular dependencies (architecture health)
curl http://localhost:7777/circular-dependency-detection-scan | jq '.data'
```

**Real Apache Iggy Result**:
```json
{"has_cycles": false, "cycle_count": 0, "cycles": []}
```

**Insight**: Zero cycles = excellent modular architecture.

```bash
# Find complexity hotspots
curl "http://localhost:7777/complexity-hotspots-ranking-view?top=10" | jq '.data.hotspots'
```

**Real Apache Iggy Patterns**:
```json
[
  {"rank": 1, "entity_key": "rust:fn:new:unknown:0-0", "inbound_count": 1523},
  {"rank": 2, "entity_key": "rust:fn:unwrap:unknown:0-0", "inbound_count": 987},
  {"rank": 3, "entity_key": "rust:fn:clone:unknown:0-0", "inbound_count": 654},
  {"rank": 4, "entity_key": "rust:fn:as_ref:unknown:0-0", "inbound_count": 543}
]
```

### 4.5 Smart Context for LLM Agents

```bash
# Get optimal context within 2000 token budget
curl "http://localhost:7777/smart-context-token-budget?focus=rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54&tokens=2000" | jq '.data'
```

**Real Response**:
```json
{
  "focus_entity": "rust:method:new:__crates_parseltongue-core_src_storage_cozo_client_rs:38-54",
  "token_budget": 2000,
  "tokens_used": 816,
  "entities_included": 8,
  "context": [
    {"entity_key": "rust:fn:Ok:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:collect:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:contains:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:default:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:map_err:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:new:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:splitn:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"},
    {"entity_key": "rust:fn:to_string:unknown:0-0", "relevance_score": 0.95, "relevance_type": "direct_callee"}
  ]
}
```

**Smart Context Algorithm**:
- Direct callers: score 1.0
- Direct callees: score 0.95
- Transitive deps: score 0.7 - (0.1 × depth)
- Greedy knapsack selection until budget exhausted

---

## 5. Entity Key Format

Entity keys follow this pattern:
```
language:entity_type:entity_name:file_path:line_range
```

**Example**: `rust:fn:authenticate:src_auth_rs:10-50`
- **Language**: `rust`
- **Type**: `fn` (function)
- **Name**: `authenticate`
- **File**: `src/auth.rs` (slashes become underscores)
- **Lines**: `10-50`

**External References**: When code calls external functions (stdlib, crate dependencies), the target has `unknown:0-0`:
- `rust:fn:new:unknown:0-0` - Stdlib `new()` calls like `HashMap::new()`
- `rust:fn:unwrap:unknown:0-0` - Stdlib `unwrap()` calls
- `rust:fn:Ok:unknown:0-0` - Result enum variant

---

## 6. Language Support

| Language | Extensions | Entity Types |
|----------|------------|--------------|
| **Rust** | `.rs` | fn, struct, enum, trait, impl, mod |
| **Python** | `.py` | def, class, async def |
| **JavaScript** | `.js`, `.jsx` | function, class, arrow functions |
| **TypeScript** | `.ts`, `.tsx` | function, class, interface, type |
| **Go** | `.go` | func, type, struct, interface |
| **Java** | `.java` | class, interface, method, enum |
| **C** | `.c`, `.h` | function, struct, typedef |
| **C++** | `.cpp`, `.hpp` | function, class, struct, template |
| **Ruby** | `.rb` | def, class, module |
| **PHP** | `.php` | function, class, trait |
| **C#** | `.cs` | class, struct, interface, method |
| **Swift** | `.swift` | func, class, struct, protocol |

---

## 7. Common Query Patterns

### 7.1 Discovery Workflow (Real Example)

```bash
# 1. Get overview (Apache Iggy: 10K+ entities)
curl http://localhost:7777/codebase-statistics-overview-summary

# 2. Check architectural health (Apache Iggy: 0 cycles ✅)
curl http://localhost:7777/circular-dependency-detection-scan

# 3. Find complexity hotspots (Apache Iggy: standard Rust patterns)
curl "http://localhost:7777/complexity-hotspots-ranking-view?top=10"

# 4. Search domain of interest (Apache Iggy: streaming = 751 entities)
curl "http://localhost:7777/code-entities-search-fuzzy?q=YOUR_SEARCH_TERM"

# 5. Analyze impact of key entities (Apache Iggy: BytesSerializable = 117 affected)
curl "http://localhost:7777/blast-radius-impact-analysis?entity=ENTITY_KEY&hops=2"
```

### 7.2 Impact Analysis Before Changes

```bash
# What functions call this function?
curl "http://localhost:7777/reverse-callers-query-graph?entity=rust:fn:function_name:file.rs:10-50"

# What does this function call?
curl "http://localhost:7777/forward-callees-query-graph?entity=rust:fn:function_name:file.rs:10-50"

# What's the blast radius of changing this?
curl "http://localhost:7777/blast-radius-impact-analysis?entity=rust:fn:function_name:file.rs:10-50&hops=3"
```

### 7.3 Code Exploration

```bash
# List all entities
curl http://localhost:7777/code-entities-list-all

# List only functions
curl "http://localhost:7777/code-entities-list-all?entity_type=function"

# Get detailed view of specific entity
curl "http://localhost:7777/code-entity-detail-view?key=ENTITY_KEY"

# Search for entities by name
curl "http://localhost:7777/code-entities-search-fuzzy?q=search_term"
```

### 7.4 LLM Context Optimization

```bash
# Get smart context within token budget
curl "http://localhost:7777/smart-context-token-budget?focus=ENTITY_KEY&tokens=4000"

# Find hidden dependencies
curl "http://localhost:7777/temporal-coupling-hidden-deps?entity=ENTITY_KEY"

# Get semantic clusters (modules)
curl http://localhost:7777/semantic-cluster-grouping-list
```

### 7.5 Real-World Insights from Apache Iggy

**Architecture Health Indicators**:
- **Zero Circular Dependencies** ✅ - Excellent modularity
- **High Entity Count** (10,362) - Large but manageable codebase
- **Multi-Language Support** (5 languages detected) - Good SDK coverage

**High-Impact Entities**:
```bash
# Core protocol trait (affects 117 entities)
curl "http://localhost:7777/blast-radius-impact-analysis?entity=rust:trait:BytesSerializable:...&hops=2"

# Central shard system (17 impl blocks)
curl "http://localhost:7777/code-entities-search-fuzzy?q=IggyShard"

# Permission system (67 entities)
curl "http://localhost:7777/code-entities-search-fuzzy?q=Permissioner"
```

**Domain-Specific Entity Counts**:
```bash
# Search streaming infrastructure
curl "http://localhost:7777/code-entities-search-fuzzy?q=streaming" | jq '.data.total_count'
# Real result: 751 entities

# Search message handling
curl "http://localhost:7777/code-entities-search-fuzzy?q=message" | jq '.data.total_count'
# Real result: 1,339 entities
```

---

## Response Format

All endpoints return consistent JSON:

```json
{
  "success": true,
  "endpoint": "/blast-radius-impact-analysis",
  "data": {
    "source_entity": "rust:fn:process:src_lib_rs:1-20",
    "total_affected": 14,
    "by_hop": [{"hop": 1, "count": 5, "entities": [...]}]
  },
  "tokens": 234
}
```

The `tokens` field helps LLMs understand context budget impact.

---

**Parse once, query forever.**

*Parseltongue: Making LLMs reason about code with graphs, not text.*