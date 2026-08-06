# Day 23 — Multi-Tool HR Agent: Tools & Descriptions

## Overview
This document lists all tools connected to the AI Agent, along with the exact description text used for each — the description is what the agent reads to decide when to use a given tool.

---

## Tool 1: Knowledge Base Search (Simple Vector Store)

**Type:** Vector Store Tool (Retrieve Documents as Tool for AI Agent)

**Purpose:** Answers HR policy questions using the company's HR document knowledge base (leave policy, working hours, attendance, internship guidelines, code of conduct).

**Description used:**
```
Searches the company HR policy knowledge base covering leave policy, working hours, attendance, internship guidelines, and code of conduct. Use this to answer any employee question about HR rules or policies.
```

**Example trigger question:** "How many casual leaves are allowed?"

---

## Tool 2: Public Holiday Checker (HTTP Request Tool)

**Name:** `check_public_holiday`

**Type:** HTTP Request Tool (GET)

**Purpose:** Checks whether a specific date is a public holiday in a given country, using the free public Nager.Date API.

**Endpoint:**
```
https://date.nager.at/api/v3/PublicHolidays/{{ $fromAI("year", "The 4-digit year to check, e.g. 2026", "string") }}/{{ $fromAI("countryCode", "2-letter ISO country code, e.g. PK for Pakistan or US for United States", "string") }}
```

**Description used:**
```
Checks whether a specific date is a public holiday. Uses an international holiday API that only covers a limited set of countries (mainly Europe, North America, and a few others — Pakistan is NOT supported). If the API returns no data or an error for the requested country, tell the user this data isn't available for that country — do not retry the same call multiple times.
```

**Example trigger questions:**
- "Is 25 December 2026 a public holiday in the United States?" → returns real data
- "Is 14 August 2026 a public holiday in Pakistan?" → gracefully falls back (unsupported country)

---

## Tool 3: Leave Request Logger (Call n8n Workflow Tool)

**Type:** Call n8n Workflow Tool → calls a separate sub-workflow ("Tool – Log Leave Request")

**Purpose:** Logs a new employee leave request to a local CSV file (`leave_requests_log.csv`), appending each new request as a new row.

**Inputs required (filled automatically by the AI Agent):**
- `employeeName` (string)
- `leaveType` (string)
- `leaveDates` (string)

**Description used:**
```
Logs a new leave request to the company's leave request file. Use this ONLY after the user has explicitly confirmed they want to submit the request. Requires employee name, leave type, and leave dates.
```

**Sub-workflow structure:**
```
When Executed by Another Workflow
  → Read/Write Files from Disk (Read mode) — reads existing leave_requests_log.csv
  → Extract from File (Extract From Text File)
  → Code (JavaScript) — appends the new row to existing content, encodes to binary
  → Read/Write Files from Disk (Write mode) — writes updated file back to disk
```

**Human-confirmation boundary (from System Prompt):**
```
Before logging any leave request, you must first summarize the details back to the user (employee name, leave type, and leave dates) and explicitly ask: "Shall I go ahead and submit this request?"

Only call the leave-logging tool after the user clearly confirms (e.g., says "yes", "confirm", "go ahead"). If the user has not yet confirmed, do not call the tool — just ask for confirmation and wait for their response.
```

**Example trigger flow:**
1. User: "I want to apply for casual leave from 10 to 12 August. My name is Ahmed."
2. Agent: summarizes details, asks "Shall I go ahead and submit this request?"
3. User: "Yes, go ahead."
4. Agent: calls the tool, confirms submission, logs the row to the CSV.

---

## Memory

**Type:** Simple Memory (Window Buffer Memory)

**Purpose:** Allows the agent to recall earlier parts of the same conversation (e.g., the user's name mentioned earlier), separate from the HR knowledge base.

**Configuration:**
- Session ID: tied to the Chat Trigger's session
- Context Window Length: 5 (last 5 exchanges retained)

---

## Full System Prompt (combined)

```
You are an HR Policy Assistant for the company.

For any question about company HR policies (leave, working hours, attendance, internship guidelines, code of conduct), you must:
1. Always use the knowledge base tool to search for relevant information first.
2. Answer ONLY using information found in the retrieved context. Do not use outside knowledge or make assumptions.
3. If the retrieved context does not contain the answer, respond exactly with: "This information is not available in the knowledge base. Please check with HR directly."
4. Briefly mention which policy area the answer came from (e.g., "According to the Leave Policy...").

For general conversational questions that are not about HR policy (e.g., the user's name, something they said earlier in the conversation), answer normally using the conversation history — do not search the knowledge base for these, and do not apply the fallback rule to them.

Before logging any leave request, you must first summarize the details back to the user (employee name, leave type, and leave dates) and explicitly ask: "Shall I go ahead and submit this request?"

Only call the leave-logging tool after the user clearly confirms (e.g., says "yes", "confirm", "go ahead"). If the user has not yet confirmed, do not call the tool — just ask for confirmation and wait for their response.

Never invent or guess company policy details. Keep answers clear, concise, and professional.
```

---

## Known Limitations
- The vector store is in-memory (experimental) and resets on n8n restart — the ingestion workflow must be re-run afterward.
- The public holiday API only covers a limited set of countries; unsupported countries correctly trigger a graceful fallback rather than a retry loop.
- The leave-logging sub-workflow must be **Published/Active** in n8n for the main agent to call it successfully.
