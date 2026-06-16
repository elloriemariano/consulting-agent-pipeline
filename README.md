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
The biggest architectural leap. Replaced the hardcoded 
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
USC Marshall School of Business | Applied Analytics Minor
Built: June 2026