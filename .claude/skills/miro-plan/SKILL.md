---
name: miro-plan
description: Use when someone asks to read a Miro board, extract a process map, create an implementation plan from a Miro board, understand the project flow from Miro, or use a Miro board to guide development. Triggered with /miro-plan [board-id-or-url].
argument-hint: [board-id-or-url]
disable-model-invocation: true
---

## What This Skill Does

Reads a Miro board, extracts the process flow/architecture, displays it as a structured step-by-step guide, and produces an implementation plan — pausing for user confirmation before any coding begins.

## Steps

### Step 1: Extract the Board ID

If  is a full Miro URL (e.g. `https://miro.com/app/board/uXjVGuCunBs=/`), extract the board ID from the URL path segment between `/board/` and `/`. If  is already a bare ID, use it directly.

If no argument was provided, ask the user: "Please provide your Miro board ID or URL."

### Step 2: Get Project Summary

Call `context_get_board_docs` with:
- `board_id`: the extracted board ID
- document type: `project_summary`

This returns a high-level overview of the board and recommends which detailed document types are most valuable for this project. Read it carefully — it determines what to fetch next.

### Step 3: Get Detailed Documentation

From the summary in Step 2, identify the applicable detailed document types (e.g. `process_flow`, `architecture`, `data_model`, `api_spec`, `user_stories`, etc.).

Call `context_get_board_docs` for each applicable type — do this in parallel where possible. Use the same `board_id`.

### Step 4: Save Docs to /docs/

Before writing any files, check if the `docs/` folder already contains files matching the document types you are about to save.

- If `docs/` files already exist: warn the user and ask for confirmation before overwriting.
- If `docs/` is empty or files don't exist: proceed directly.

Save each document as `docs/[document-type].md` (e.g. `docs/project_summary.md`, `docs/process_flow.md`).

Also update or create `docs/README.md` with a table of contents linking to each saved file and a one-line description of each.

### Step 5: Render the Process Map

Using the fetched documentation, render the project's process flow as a structured map directly in the conversation. Use this format:

```
## Process Map: [Project Name]

### Phase 1: [Phase Name]
1. [Step description]
2. [Step description]
   → Decision: [condition] → [outcome A] / [outcome B]
3. [Step description]

### Phase 2: [Phase Name]
...

### Key Decision Points
- [Decision]: [Branch A] vs [Branch B]

### Success Criteria
- [What "done" looks like for the full flow]
```

Be specific — use the actual names, nodes, and logic from the board. Do not paraphrase vaguely.

### Step 6: Present Plan and Pause

After displaying the process map, present a proposed implementation order:

```
## Proposed Implementation Plan

**Phase 1 — [Name]**
- Task: [specific task]
- Deliverable: [what gets built]
- Acceptance criteria: [how to verify it works]

**Phase 2 — [Name]**
...
```

Then ask: **"Does this plan look correct? Should I begin implementation, or are there changes to make first?"**

Do NOT write any code until the user explicitly confirms. Wait for their response.

### Step 7: Begin Implementation (on confirmation)

Once the user confirms, begin implementation phase by phase:
- Complete one phase fully before moving to the next
- After each phase, demonstrate the working slice (run tests, show output, or describe what was built)
- Check in with the user between phases

## Notes

- Never start coding without explicit user confirmation after Step 6
- Never overwrite existing `docs/` files without warning the user first
- If the board has no structured process flow, describe what IS present and ask the user to clarify what they want extracted
- If `context_get_board_docs` returns an error, report the error message clearly and ask the user to verify the board ID and that the Miro MCP server is connected
- Keep the process map in the conversation concise — detailed content lives in `docs/`
