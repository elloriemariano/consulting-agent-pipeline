## About This Project

This project was developed as a hands-on exploration of agentic AI workflows, dynamic orchestration, and prompt engineering. The goal was not to build a production-ready system, but rather to better understand how specialized AI agents can collaborate to solve complex business problems.

The implementation demonstrates core concepts such as agent specialization, orchestration, context design, human-in-the-loop validation, and structured output generation. Several components are intentionally simplified, and there are many opportunities for future enhancements, including expanded retrieval capabilities, more robust evaluation frameworks, and production-grade deployment patterns.

The project is best viewed as a prototype used to explore how multi-agent architectures may support business decision-making workflows.

# Consulting Agent Pipeline

An AI-powered multi-agent consulting pipeline built with the 
Anthropic Claude API and Python. Takes any business problem 
and automatically produces a structured consulting brief 
through specialized AI agents.

Built in one evening as a hands-on exploration of agentic 
AI architecture, dynamic orchestration, and context design.

---

## The Build Story

### Version 1 — Foundation (3 agents, fixed pipeline)
Started with three agents in a hardcoded sequence:
Problem Analyst → Solution Architect → Risk Reviewer.
It worked, but had clear limitations.

**Limitations identified:**
- Fixed pipeline — same agent order regardless of problem complexity
- No input validation — vague problems produced shallow outputs
- No quality checks — Agent 2 blindly trusted Agent 1's output
- Missing specializations — no financial modeling, no change management
- No executive output — no C-suite summary

### Version 2 — Addressing Limitations
Iterated on each limitation:
- Added **Intake Agent** with human-in-the-loop clarification
- Added **quality validation** between every agent
- Added **Executive Summarizer** for C-suite one-pagers
- Added **formatted timestamped reports** to outputs folder

### Version 3 — Dynamic Orchestration
Replaced the hardcoded 
sequence with a true orchestrator agent — an LLM that reads 
each problem and dynamically decides which agents to deploy.

Added two new specialist agents:
- **Financial Modeler** — deployed when problem mentions 
  budget, cost, or ROI
- **Change Management** — deployed when problem mentions 
  adoption, resistance, or organizational change

Result: the same pipeline deploys 4 agents for a simple 
problem and all 6 for a complex enterprise transformation.

---

## Agent Architecture
User inputs business problem

↓

Intake Agent

(validates completeness, asks clarifying questions,

human-in-the-loop checkpoint)

↓

Orchestrator Agent

(reads problem, dynamically selects which agents

to deploy and in what order)

↓

┌─────────────────────────────────┐

│  Problem Analyst                │

│  Solution Architect             │

│  Risk Reviewer                  │

│  Financial Modeler (if needed)  │

│  Change Management (if needed)  │

└─────────────────────────────────┘

↓

Executive Summarizer

(C-suite one-pager from all outputs)

↓

Formatted timestamped brief

saved to outputs/ folder


### The 8 Agents

**Intake Agent** — Evaluates whether the problem statement 
is complete enough for analysis. If not, asks up to 3 
specific clarifying questions and waits for user input 
before proceeding. Human-in-the-loop checkpoint.

**Orchestrator** — Reads the enriched problem and outputs 
a JSON deployment plan selecting which specialist agents 
to call and in what order. Always deploys problem analyst 
first and executive summarizer last.

**Problem Analyst** — Breaks down the core pain point, 
identifies all stakeholders affected, maps current vs 
desired state, and surfaces the 3 key questions that 
must be answered before solution design begins.

**Solution Architect** — Designs a practical AI-powered 
solution with technical approach (RAG, agents, LLMs, 
automation), three implementation phases (pilot, scale, 
optimize), and quantified business impact.

**Risk Reviewer** — Stress-tests the proposed solution. 
Identifies the top 3 risks that could kill the project, 
gaps the solution missed, and specific recommendations 
to strengthen the approach before client presentation.

**Financial Modeler** — Builds a detailed ROI model with 
implementation costs (low/high ranges), annual benefits 
broken into labor savings and revenue impact, payback 
timeline, and a downside sensitivity scenario. Flags 
every assumption explicitly.

**Change Management** — Addresses the human side of AI 
adoption. Names specific stakeholder groups and their 
resistance drivers, identifies what would actually kill 
the project culturally, and produces a concrete enablement 
plan with leading indicators of adoption success.

**Executive Summarizer** — Produces a 200-word C-suite 
brief: situation, opportunity, recommendation, three 
quantified outcomes, top risk, and immediate next step. 
Clean enough for a CEO to read in 90 seconds.

---

## Key Design Decisions

**MD files as agent context**
Each agent's role, expertise, constraints, and output 
format are defined in a dedicated markdown file. The same 
underlying Claude model produces completely different 
specialized outputs based purely on the context in each 
MD file. Context quality is the quality ceiling — not 
model capability.

**Dynamic orchestration**
The Orchestrator is itself an LLM that reasons about 
the problem and decides which agents to deploy. A simple 
automation request might need 4 agents. A complex 
enterprise transformation with budget constraints and 
change management needs all 6. The pipeline adapts to 
problem complexity rather than applying a one-size-fits-all 
sequence.

**Human-in-the-loop**
The Intake Agent pauses the pipeline when the problem 
statement is incomplete and asks specific clarifying 
questions. Agents always work from rich, validated context 
rather than guessing from vague inputs.

**Quality validation**
Each agent's output is checked for required sections 
before being passed to the next agent. Missing sections 
are flagged and logged rather than silently propagated 
through the pipeline.

**Iterative development**
The pipeline evolved through three versions — each 
addressing specific limitations identified in the 
previous version. This mirrors how production AI systems 
are actually built: start simple, identify failure modes, 
iterate.

---

## Resilience & Error Handling

**Retry with backoff**
Every agent call goes through the Anthropic API, which can fail
transiently — rate limits, timeouts, connection drops, server-side
overload. `call_agent` retries these specific errors up to 3 times
with exponential backoff (2s, 4s, 8s) before giving up. Non-retryable
errors (bad request, authentication failure) fail immediately instead
of wasting retries on something that will never succeed.

**Partial save on failure**
Each agent call is a paid, completed piece of work — losing it because
a *later* agent failed would throw away real progress. If an agent
exhausts its retries, the pipeline doesn't crash with nothing to show:
it assembles a brief from whatever sections did complete and saves it
to `outputs/` as `partial_brief_*.md` (instead of `brief_*.md`), with
the failure reason recorded in the file's frontmatter (`status: partial`,
`failure_reason: ...`) so it's clear afterward what ran and what didn't.

---

## Claude Code Integration

This repo is set up to work with [Claude Code](https://claude.ai/code) as a
development partner, not just for writing the pipeline itself.

**`CLAUDE.md`**
A root-level context file that gives Claude Code an architectural map of the
repo on every session — the four-stage pipeline flow (intake → orchestrator →
dynamic agent loop → assembly), the markdown-as-agent-prompt pattern, the
coupling between `agents/orchestrator.md`'s agent list and `validation_map`
in `main.py`, and the steps to follow when adding a new specialist agent.

**`pipeline-reviewer` skill**
A custom skill at `.claude/skills/pipeline-reviewer.md` that audits the
pipeline's architecture on demand. When invoked, it reads `main.py` and
every file in `agents/`, cross-checks them against each other (orchestrator's
deployable agent list vs. actual files, `validation_map` coverage vs. each
agent's declared output sections, failure-mode handling), and produces a
structured critique: an overall score out of 10, top 3 strengths, top 5
weaknesses each with a specific recommendation, and a prioritized action
list of what to build next.

To invoke it in a Claude Code session, ask Claude to "run the
pipeline-reviewer skill" (or reference it directly as `/pipeline-reviewer`
once registered for the session).

The skill has been run twice so far, tracking real architectural progress:
- **Initial review: 6/10** — flagged cosmetic output validation, no error
  handling around the Anthropic API call, silent orchestrator fallbacks,
  a one-size-fits-all token budget, and silently-skipped agents.
- **After resilience improvements: 7/10** — confirmed the retry/backoff
  and partial-save changes (see *Resilience & Error Handling* above)
  resolved the top-priority weakness for transient API failures, while
  surfacing one residual gap (non-retryable errors, like a bad API key,
  still bypass the partial-save path) plus the four other weaknesses
  that remain open for future iterations.

---

## Tech Stack

- Python 3.11
- Anthropic Claude API (claude-haiku-4-5)
- python-dotenv for environment management
- No external frameworks — pure Python orchestration

---

## Setup

1. Clone the repo
2. Install dependencies:
```bash
pip3 install anthropic python-dotenv
```
3. Create `.env` file: ANTHROPIC_API_KEY=your_key_here

4. Run:
```bash
python3 main.py
```

---

## Tested Industries

| Industry | Problem | Agents Deployed |
|----------|---------|-----------------|
| Construction | $2B portfolio cost overrun detection | 4 |
| Customer Service | 2000 inquiries/week, 4-day response time | 6 |
| Logistics | 300 freight invoices/day, 12% error rate | 6 |
| Healthcare | Hospital scheduling, 500 appts/day | 6 |
| Sales | Follow-up email automation, 50 reps | 6 |

---

## What I Learned

The biggest insight from building this: **context is 
everything in agentic AI systems**. The model capability 
is rarely the bottleneck. What determines output quality 
is how precisely you define each agent's role, constraints, 
and output format in its MD context file.

The second insight: **a fixed pipeline is not an agent**. 
True agentic behavior requires the orchestrator to reason 
about what to do next based on what it observes — not 
follow a predetermined script. The jump from v1 to v3 
was the jump from a prompt chain to a genuine agent system.

---

## Built By

Ellorie Mariano
USC Marshall School of Business | Applied Analytics & Web Development Minors
Built: June 2026
