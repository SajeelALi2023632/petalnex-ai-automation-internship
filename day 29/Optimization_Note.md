# Day 29 — Optimization Note

**Workflow optimized:** `day 16/My workflow 13.json` (Gmail email classifier & router)
**Source:** github.com/SajeelALi2023632/petalnex-ai-automation-internship

## What it does
Gmail Trigger → LLM classifies each email (category, priority, sentiment, department, summary) → `If`/`Switch` routes it into Sales / Support / Complaint / Invoice / Spam / General / Fallback.

## The problem: a redundant AI call
The `Structured Output Parser` node had `autoFix: true`. Whenever the classifier's raw text didn't parse as valid JSON on the first try, n8n silently fired a **second `gpt-5-mini` call** (`OpenAI Chat Model1`) just to reformat the first model's output into valid JSON.

That's a full model invocation spent solely on JSON formatting — something deterministic code does instantly, for free, with zero API latency.

## The fix
- Removed `OpenAI Chat Model1` and `Structured Output Parser` entirely.
- Turned off `hasOutputParser` on `Basic LLM Chain` and tightened its prompt to explicitly request raw JSON only.
- Added a new **`Parse & Validate Classification`** Code node that:
  - Strips markdown code fences if the model wrapped the JSON in them.
  - Tries `JSON.parse`, then falls back to regex-extracting the first `{...}` block and parsing that.
  - Validates the result against the same schema (`category`/`priority`/`sentiment` enums, `department`/`summary` strings).
  - If validation still fails, deterministically emits a safe default (`category: "Unknown"`, routed for manual review) — matching the old fallback behavior — instead of retrying with another LLM call.
  - Logs the outcome (`console.log`) either way, satisfying the "sensible logging" requirement.
- Rewired `Basic LLM Chain → Parse & Validate Classification → If`, leaving the rest of the routing logic (`If`/`Switch`/Sales/Support/etc.) untouched.

## Before / After

| | Before | After |
|---|---|---|
| AI calls per email (happy path) | 1 | 1 |
| AI calls per email (malformed JSON path) | 2 (classification + autoFix) | 1 (classification only) |
| Nodes | 14 | 13 |
| JSON repair method | Second LLM call | Deterministic Code node (regex + `JSON.parse`) |
| Cost/latency on malformed output | +1 full model round-trip | +0 (near-instant local parse) |
| Logging | None | `console.log` on both success and failure paths, including raw output on failure for debugging |

**Net effect:** identical behavior on the happy path, and up to a 50% reduction in AI calls (and the associated cost + latency) on the subset of emails where the model's first response wasn't perfectly formatted — with better debuggability than before, since failures now log the offending raw text instead of quietly retrying.

## Files
- `original_workflow.json` — unmodified copy of `day 16/My workflow 13.json`
- `optimized_workflow.json` — the optimized version, ready to import into n8n
