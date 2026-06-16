# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file Python pipeline (`main.py`) that orchestrates multiple Claude API calls to turn a raw business
problem statement into a structured, multi-section consulting brief. There is no framework — orchestration,
prompting, validation, and file output are all plain Python in `main.py`. Each "agent" is just a system prompt
stored as a markdown file in `agents/`.

## Commands

```bash
# Install dependencies (no requirements.txt — install directly)
pip3 install anthropic python-dotenv

# Run the pipeline (interactive — prompts for a business problem on stdin)
python3 main.py
```

There are no tests, linter, or build step in this repo.

Requires a `.env` file with `ANTHROPIC_API_KEY=<key>` (loaded via `python-dotenv`). `.env` is gitignored.

## Architecture

The pipeline runs as a strict sequence of stages inside `run_pipeline()` in `main.py`:

1. **Intake (`run_intake`)** — calls `agents/intake_agent.md` to check whether the problem statement has
   enough detail (core problem, scale, current process, quantified impact, desired outcome). If the agent's
   first response line is `NEEDS_CLARIFICATION`, the pipeline blocks on `input()` for human-in-the-loop
   answers before continuing. If `READY`, it proceeds unchanged.

2. **Orchestration (`run_orchestrator`)** — calls `agents/orchestrator.md`, an LLM acting as "managing
   director" that reads the enriched problem and returns a JSON plan: `{"agents": [...], "reasoning": "..."}`.
   This determines *which* specialist agents run and in *what order* — the pipeline is not hardcoded.
   JSON is extracted by finding the first `{` and last `}` in the response (tolerates stray text/backticks).
   If parsing fails for any reason, it falls back to a fixed default sequence (problem_analyst →
   solution_architect → risk_reviewer → executive_summarizer).

3. **Agent execution (`run_agent_by_name`, looped in `run_pipeline`)** — runs each agent named in the
   orchestrator's plan, in order. Each agent receives the full accumulated context: the original problem
   plus every prior agent's output concatenated as `previous_outputs` (the string just keeps growing).
   After each call, `validate_output` checks the result against a per-agent list of required section
   keywords (see `validation_map` in `run_agent_by_name`) and only logs a warning on mismatch — it never
   blocks or retries.

4. **Assembly (`save_formatted_report`)** — concatenates outputs into a final markdown brief (executive
   summary first if present, then other sections in orchestrator order), adds a YAML-ish frontmatter header,
   and writes a timestamped file to `outputs/`.

### Agent definitions (`agents/*.md`)

Each specialist is defined purely by its markdown file, passed as the `system` prompt to a single
`client.messages.create` call (see `call_agent` in `main.py`) using `claude-haiku-4-5-20251001`. There is no
agent code — adding or changing a specialist's behavior means editing its `.md` file, not Python. Each file
generally follows the same internal structure: Role, Task/Decision Rules, Output Format, Constraints.

Available agents and the orchestrator's rules for deploying them (`agents/orchestrator.md`):
- `problem_analyst` — always first
- `solution_architect` — always included
- `risk_reviewer` — always included for client-facing/high-stakes problems
- `financial_modeler` — only if the problem mentions budget/cost/ROI/savings/revenue
- `change_management` — only if the problem mentions adoption/resistance/training/culture
- `executive_summarizer` — always last

### Adding a new specialist agent

1. Create `agents/<name>.md` following the existing Role/Task/Output Format structure.
2. Add an entry to `validation_map` in `run_agent_by_name` (main.py) listing required output sections.
3. Update `agents/orchestrator.md`'s "Available Agents" list and decision rules so the orchestrator knows
   when to deploy it.
4. Optionally add a friendlier title to `section_titles` in `run_pipeline` for the final brief.

### Key coupling to be aware of

- The orchestrator's JSON `agents` array values must exactly match filenames in `agents/` (minus `.md`) —
  there's no normalization step.
- `run_agent_by_name` special-cases the user-message construction for `problem_analyst` (gets only the raw
  problem) and `executive_summarizer` (gets the full assembled brief framed as "summarize this"); all other
  agents get the generic "original problem + previous outputs" framing.
- `outputs/` accumulates every run's brief as a separate timestamped file — nothing is cleaned up
  automatically.
