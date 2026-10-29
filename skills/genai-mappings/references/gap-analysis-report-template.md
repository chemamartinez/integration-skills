# GenAI field gap analysis report template

Follow this structure. Fill every section. Where a section does not apply, write `N/A` and one
sentence saying why.

---

```markdown
# ECS GenAI Field Mapping: {{INTEGRATION_NAME}}

## Summary

[3–5 sentences: what the integration collects, which streams are in scope, the most important
gaps, and what must change at the source (opt-ins, transports, new streams).]

Counts cover the core `gen_ai.*` fields only. Mixed streams get one row per record-type group.

| Stream | Record types | Kind | ✅ | ⚠️ | 📥 | 🔌 | 🔄 | ⛔ | — |
|---|---|---|---|---|---|---|---|---|---|
| `<stream>` | `<types, or "all">` | Inference / Agent / Tool execution / Message / Usage metrics / Product telemetry | n | n | n | n | n | n | n |

## ECS pin

| Current `build.yml` pin | Pipeline `ecs.version` | Required | Action |
|---|---|---|---|
| `git@v8.11.0` | `8.11.0` | `git@v9.5.0` | Bump both |

## Data stream classification

| Stream | Record type | Kind | Deciding evidence |
|---|---|---|---|
| `<stream>` | `<event type or "all">` | ... | [endpoint, field or fixture] |

## Content capture and source configuration

What the user must enable at the source for content fields to exist, and what the README says
today.

| Setting | Controlled by | Enables | Limits | In README? |
|---|---|---|---|---|
| `OTEL_LOG_RAW_API_BODIES=1` | user's environment | `input.messages`, `output.messages` | — | yes / no |

## Field mapping: `<stream>` (repeat per in-scope stream)

Single-kind stream:

| Field | Status | Source | Notes |
|---|---|---|---|
| `gen_ai.provider.name` | 🔄 | constant | `aws.bedrock` for every record |
| `gen_ai.operation.name` | 🔄 | `operation` | Only in `event.action` today; map `Converse`→`chat` |
| `gen_ai.request.model` | 📥 | `modelId` | |
| ... every field from the workflow's Phase 4 list ... | | | |

Mixed stream: one status column per record-type group. Each cell is `<status> <source>`, and
notes go below the table.

| Field | `api_request` | `api_request_body` | `tool_result` |
|---|---|---|---|
| `gen_ai.request.max_tokens` | — (on `api_request_body`) | 🔌 `body.max_tokens` | — |
| `gen_ai.tool.name` | — | — | ✅ `tool_name` |

### ⚠️ Incorrect mappings

For each:
- **Field:** `gen_ai.x`
- **Now:** [what the package does]
- **Should be:** [per the catalog]
- **Breaking?** [does fixing it remove or retype a published field?]

### 📥 / 🔌 Gaps

For each:
#### `gen_ai.x`
- **Layer:** 2 (received) / 3 (source)
- **Source:** [exact path in the payload, or the vendor doc URL]
- **What is needed:** [parse an existing field / enable setting X / call endpoint Y / switch transport / new data stream]
- **Notes:** [formats, size limits, string vs object. Describe the data, not processors.]

### ⛔ Not available

| Field | Layer 1 | Layer 2 | Layer 3 |
|---|---|---|---|
| `gen_ai.request.seed` | not mapped | not in any fixture | API has no seed parameter ([doc]) |

## Values with no ECS field

Kept in the vendor namespace. Never mapped to a `gen_ai.*` name until a released ECS version
defines it.

| Value | Stream | Source | Vendor field | Status | ECS field once released |
|---|---|---|---|---|---|
| cache read tokens | | `usage.cache_read_input_tokens` | `<integration>.<stream>.cache_read_input_tokens` | | — (OTel `gen_ai.usage.cache_read.input_tokens`, not in ECS) |
| session id | | `session.id` | `<integration>.<stream>.session_id` | | `gen_ai.conversation.id` (ECS `main`) |

## Adjacent ECS fields

| Field | Stream | Source | Status |
|---|---|---|---|
| `file.path` | | tool arguments `file_path` | |

## Differences from the tracking issue

| Field | Issue says | Finding | Evidence |
|---|---|---|---|
| `gen_ai.provider.name` | `az.ai.openai` | `azure.ai.openai` | OTel registry |

## Documentation changes

[What the README must add: opt-ins, required settings, what is collected with and without them.]

## Open questions

[Decisions for the team: breaking changes, new streams or transports, ambiguous provider or
operation values.]
```
