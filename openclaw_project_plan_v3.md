# Demystify OpenClaw with Tinkering
## The Architecture of Autonomy

**NinjaLABO Intern Research Project**
Nghi Vo Dong / Supervisor: Hiroshi
Duration: 11 weeks / 10h per week = about 110 hours

---

## Project Purpose

> **If NinjaLABO designs a future vertical PA (Personal Agent), what should we build, and what should we deliberately avoid building?**

This is not a project to build a new Agent from scratch.
It is a project to use existing LLM-based Agents, **OpenClaw / Nanobot**, as **observation targets**, and to build notebook-based measurement infrastructure that can **quantitatively measure** where and why the process breaks when **LLM reasoning automatically mashes up multiple APIs to complete higher-level tasks**. The goal is to produce a data-driven map of **where the boundary lies between LLM control and code control**.

**What this project will build:**
- A **log schema** to record Agent behavior
- **Measurement infrastructure** (`kpi_collector.py`) to aggregate KPIs automatically
- An **evaluation task set** (`tasks/baseline.json`) composed from existing benchmarks
- A reproducible set of **notebooks** for experiments, analysis, and visualization

**What this project will not build:**
- The core code of OpenClaw / Nanobot / OpenFang
- A new Agent framework
- Any hand-written control flow / orchestration / retry policy inserted before observing the reasoning-first baseline
- A production PA product

The final deliverable is not a paper. It is a **boundary map grounded in measured data** that Hiroshi can use for future design decisions.

---

## Principles

> **All work must be executed, implemented, analyzed, and documented inside nbdev notebooks in a reproducible form.**

```text
experiment -> in notebook code
implement  -> in notebook code
document   -> in prose inside the same notebook
publish    -> as blog posts generated from the notebook
```

The artifact unit is the notebook, and the toolchain is nbdev.
Document or script work done outside notebooks without the nbdev toolchain does not count as a deliverable.
However, Python modules generated via `nbdev export`, and blog posts generated from notebooks (GitHub Pages), are valid derived deliverables.

**Definition of reproducibility:**
LLM experiments are inherently non-deterministic, so we do not guarantee identical outputs. We guarantee that **the same analysis can be rerun under the same experimental conditions**.

```text
Reproducibility controls:
- Record the model and version in config.py
- Use temperature=0 by default
- Record run_seed when the API supports it; otherwise record null
- Cache external API responses; decide the caching method in W1
- Report trends over N>=5 / N>=10 repeated runs, not one-off wins
- Keep weekly blog posts reproducible from the same notebook
```

**Observer notes:**
In addition to quantitative KPIs, we will record design intuition in notebook prose.
Examples:
- "LLM reasoning failed here because ..."
- "This failure pattern could have been prevented by better API design."

These notes become supporting evidence when Hiroshi makes design decisions in W11.

**LLM-first measurement rule:**

```text
1. reasoning-first baseline
   - First run the observed Agent as-is
   - We do not add new external control flow

2. soft scaffolding
   - We may add prompt / memory / skill / context shaping only
   - We do not add hand-written routers / if-else / retry controllers

3. hard fallback
   - We do not implement this in the mainline experiments
   - In W11, we only describe where fallback would be justified as a design output
```

---

## Publishing Stack

```text
nbdev notebook
(code + experiments + analysis in one place)
	|
Rendered to HTML with Quarto
	|
Published to GitHub Pages
```

**Cadence:**
- Notebook updates: weekly
- Blog publication: one post every 1-2 weeks
- Target number of posts: 11
- Minimum acceptable number of posts: 8

---

## Experimental Targets

| Role | Tool | Language | Autonomy | Positioning |
|------|------|----------|----------|-------------|
| **Target A** | OpenClaw | TypeScript | Reactive | Main comparison target, full-featured reference |
| **Target B** | Nanobot | Python | Reactive | Main comparison target, readable reference implementation |
| **Supplementary** | memU | Python | Reactive | Alternative approach to memory design |
| **Supplementary** | OpenFang | Rust | Autonomous | Comparison target for autonomous behavior |
| **Reference** | OpenXJarvis | Python | Reactive | Optional OpenClaw-related reference |

**Autonomy spectrum:**

```text
reactive   -> OpenClaw / Nanobot (triggered by user input)
autonomous -> OpenFang (operates autonomously on schedules)
```

**Comparison principles:**
- **mainline comparison:** OpenClaw vs Nanobot
- In the mainline comparison, use the **same model, same task set, same judgment criteria, and the same normalized tool inventory** so that we measure architecture differences rather than capability differences
  - Give OpenClaw / Nanobot the same external tool surface
  - In W1, define and freeze the **baseline tool inventory** that both agents can use
  - Skills introduced in W5 are managed separately as a **scaffold skill set**, so they are explicit intervention conditions and do not change the baseline conditions
- Use OpenFang only for the **supplementary comparison** in W10
- Because OpenFang may expose a different logging granularity, compare it only on the **common-KPI subset**

**Primary implementation risk in observation work:**
- OpenClaw / Nanobot / OpenFang are implemented in TypeScript / Python / Rust, so normalizing them into a shared log schema is the main technical bottleneck in W1-W3
- Start by extracting events from stdout, existing logs, and externally visible behavior at the lowest possible cost
- If needed, add thin wrappers without materially changing the internal code of the observed agents
- In W1, decide whether stdout parsing is enough or whether wrappers are necessary, then freeze the logging approach

**Reactive vs Autonomous comparison protocol (used in W10):**

```text
To keep the comparison fair, standardize:
- Input tasks:        the same baseline.json task set
- Initial conditions: as close as possible to the same inputs and observation window
- Success judgment:   the same goal_condition-based judgment
- Time windows:       1h / 3h / 6h
- Recorded differences:
  separate method differences (control structure differences)
  from condition differences (input or environment differences)
```

---

## Core Question

**Definition of LLM reasoning in this document:**
Delegating to the LLM the decision of "what to do next" that would traditionally be hard-coded with if/else or a state machine. Concretely, this includes:
1. Decomposing a task into subtasks
2. Choosing the appropriate API / tool
3. Chaining multiple APIs to achieve a higher-level task
4. Replanning after failure

> **"When LLM reasoning is allowed to carry the full control flow, where does it break? Only at those points is code fallback justified."**

**Thesis:**
The core assumption of this project is that LLM reasoning can choose and compose multiple APIs automatically and complete higher-level tasks without static control code. The research goal is to turn the failure boundary of that assumption into a quantitative boundary map.

**Hypothesis:**

```text
LLM reasoning = primary control
		(API selection, composition, task decomposition)
code          = fallback
		(only where the LLM demonstrably fails)
```

---

## Sub-Questions

1. **Control Boundary**: Assuming LLM reasoning handles control, where is code fallback actually necessary?
2. **Failure Modes**: Where do decomposition failures, tool-selection failures, and context contamination occur?
3. **Memory Efficiency**: How do memory, history, and summarization contribute to autonomy?
4. **Retry Behavior**: Under what conditions does retry occur, and what kinds of input changes improve it?
5. **Multi-Agent**: How far can collaboration extend the limit of a single agent?
6. **Reactive vs Autonomous**: What is fundamentally different about OpenFang's autonomous design?

---

## KPI Design

| KPI | What it measures | Main use |
|-----|------------------|----------|
| **Reasoning-First Success Rate** | Task success under the baseline with no added hand-written control flow | Primary boundary indicator |
| **Soft-Scaffolding Gain** | Improvement in success rate after adding prompt / memory / skill scaffolds | Measures the effect of non-code intervention |
| **Fallback Justification Rate** | Share of task families that still fail after soft scaffolding and satisfy fallback criteria | Code-fallback decision support |
| **Task Completion Rate** | Share of tasks that reach the goal condition | Main KPI |
| **Tool Selection Accuracy** | Share of steps where the correct tool was chosen | Failure factor analysis |
| **Decomposition Stability** | Consistency of subtask decomposition for the same request | W9 focus |
| **Retry Count per Task** | Average number of retries per task | W8 focus |
| **Retry Improvement** | Change in retry success before and after input intervention | W8 focus |
| **Context Degradation** | Quality drop in long-running tasks | W10 focus |
| **Hallucination Rate** | Share of runs that invoke nonexistent tools / APIs / impossible plans | Frozen in W2 |
| **Autonomous Duration** | Upper limit of autonomous operation (1h / 3h / 6h) | W10 focus |
| **Plan Length vs Failure** | How failure rate changes at 1-step / 3-step / 10-step plans | Optional |

**Separate the judgment layers:**

```text
1. tool_correct (step-level, immediate judgment)
   - Was the chosen tool appropriate at that point?
   - Even if the agent recovers later, a wrong tool choice stays wrong
   - Used for Tool Selection Accuracy

2. task_success (task-level, goal-based judgment)
   - Did the final result satisfy the goal condition?
   - A different but equally valid path still counts as success
   - Used for Task Completion Rate
```

**Slice KPIs by intervention layer:**
- `reasoning_first`: run the observed agent as-is with no new external control flow
- `soft_scaffold`: add prompt / memory / skill / context shaping only
  - Use `scaffold_type` to distinguish `memory`, `prompt_scaffold`, `multi_agent`, and `combined`
  - Here, `combined` specifically means the cumulative `memory + skill` arm used in W5
  - Use `scaffold_config_id` to record the exact intervention configuration, such as `"soul_v2"` or `"skill_file_ops"`
- `hard_fallback`: not an execution layer; used only as a design recommendation in W11

**Definition of Hallucination Rate (freeze the base definition by W2):**
- Calling a nonexistent tool name
- Assuming a nonexistent API or argument schema
- Continuing execution while assuming an impossible plan is still viable

If new categories are discovered in W7, measure them separately as **base** and **extended**.
Weekly reports must show both `hallucination_base_rate` and `hallucination_extended_rate` so time-series comparisons stay valid.

**Handling ambiguous cases:**
- Record unclear cases as `ambiguous`
- If a task has a high `ambiguous` ratio, redefine its goal condition in W1 or W2

**Goal condition design principles:**
- Make judgment as automatic as possible
- Preferred judgment types: file creation, exact or bounded text match, JSON state match, exit code, or match against a known answer set
- If a `goal_condition` requires human judgment, remove it from the baseline or demote it to a supplementary task
- In the first W1 draft of `baseline.json`, prefer "small but machine-judgeable" over "larger but ambiguous"

**Conditions under which code fallback is justified**
(*provisional in W2, recalibrated using pilot results in W5*):
- `reasoning_first_success_rate < 40%` for the same task family across `N>=5` repeated runs
- `soft_scaffolding_gain < +20pp`, meaning non-code intervention still does not recover enough performance
- A dominant failure mode is reproduced in `>=60%` of runs, so it is not accidental
- The failure point can be localized as a bounded control decision
- If these conditions are not met, treat the issue as primarily a problem of LLM reasoning / prompt design / API design, not a justification for code fallback

**Note on thresholds:**
The values `40% / +20pp / 60%` are provisional in W2 and will be calibrated using pilot runs from W1-W2.
In W5, run a sensitivity analysis by moving thresholds by `+-10pp` and checking whether the fallback-candidate set changes.
If the final conclusion depends strongly on threshold choice, state that explicitly in the W11 boundary map.

**Automate KPI measurement by W3:**

```python
# kpi_collector.py (exported from 02_core_loop_trace.ipynb)
# Takes logs as input, aggregates all KPIs automatically,
# and visualizes them with Matplotlib / Plotly
```

---

## Fixed Experimental Conditions (Agreed in W1)

```python
# config.py (exported from 00_setup.ipynb)
MODEL      = "claude-sonnet-4-6"   # mainline comparison uses the same model across tools; capacity ablations are separate
TASK_SET   = "tasks/baseline.json"
LOG_FORMAT = "json"

API_BUDGET_EXPECTED_USD = 30
API_BUDGET_HARD_MAX_USD = 50

# In W1, estimate per-experiment cost and verify that the total stays within $30-50
# If cost exceeds the limit, degrade in this order:
# 1. Drop the Plan Length experiment
# 2. Shorten W10 6h runs to 3h
# 3. Reduce W9 from N=10 to N=5
# 4. Limit OpenFang comparison to the common-KPI subset
# 5. Skip W6 Multi-Agent
```

---

## Log Schema (Established in W1-W2)

By the end of W2, define this formally as JSON Schema and reference it from `kpi_collector.py` in W3.

```python
# log_schema.py (exported from 01_core_loop_read.ipynb)
{
  # --- Identifiers ---
  "task_id":              str,
  "run_id":               str,
  "session_id":           str,
  "run_seed":             int | null,

  # --- Experimental metadata ---
  "agent":                str,   # "openclaw" | "nanobot" | "openfang"
  "timestamp":            str,   # ISO 8601
  "phase":                str,   # "reasoning" | "tool_call" | "observation" | "replanning" | "final"
  "autonomy_type":        str,   # "reactive" | "autonomous"
  "intervention_layer":   str,   # "reasoning_first" | "soft_scaffold"
  "scaffold_type":        str | null,   # "memory" | "prompt_scaffold" | "multi_agent" | "combined"
  "scaffold_config_id":   str | null,   # e.g. "soul_v2", "skill_file_ops"
  "model":                str,

  # --- I/O ---
  "input":                str | null,
  "output":               str | null,

  # --- Step-level evaluation ---
  "tool_used":            str | null,
  "tool_correct":         str | null,   # "correct" | "incorrect" | "ambiguous"
  "retry_count":          int | null,
  "retry_origin":         str | null,   # "llm_self_repair" | "prompt_scaffold" | "agent_builtin"

  # --- Task-level evaluation ---
  "task_success":         bool | null,
  "goal_condition_id":    str | null,

  # --- Decomposition data ---
  "plan_length":          int | null,
  "decomposition":        list[str] | null,
  "decomposition_hash":   str | null,

  # --- Error data ---
  "hallucination_base":   bool | null,
  "hallucination_ext":    bool | null,
  "error_type":           str | null
}
```

**Nullable rules:**
- `tool_used`, `tool_correct` only have values when `phase="tool_call"`
- `decomposition` only has a value on the initial `reasoning` phase
- `task_success` only has a value when `phase="final"`
- `retry_origin` only has a value when `retry_count > 0`
- `scaffold_type`, `scaffold_config_id` only have values when `intervention_layer="soft_scaffold"`
- `run_seed` is `null` when the API does not support seeding

---

## Benchmark Strategy

`baseline.json` is not fully hand-written. It is composed as a subset extracted from existing benchmarks.
The goal is to maximize **comparability** and **reproducibility**.

**Initial candidates:**

| Benchmark | Target count | What it mainly measures |
|-----------|--------------|-------------------------|
| **BFCL** | 30-50 | Tool Selection Accuracy, Hallucination Rate |
| **GAIA** Level 1-2 | 15-20 | Task Completion Rate, Decomposition Stability |
| **tau-bench** | 20-30 | Retry Behavior, Context Degradation |

**KPI-to-benchmark mapping:**

| KPI | Main benchmark | Why |
|-----|----------------|-----|
| Task Completion Rate | GAIA, tau-bench | Easy to judge task success |
| Tool Selection Accuracy | BFCL | Best suited for direct tool-choice measurement |
| Decomposition Stability | GAIA | Requires multi-step reasoning |
| Retry Behavior | tau-bench | Good for multi-turn failure and retry observation |
| Hallucination Rate | BFCL | Easy to capture improper tool invocation |
| Context Degradation | tau-bench | Better for longer interactions |

**Relation of the new KPIs to benchmarks:**
Reasoning-First Success Rate / Soft-Scaffolding Gain / Fallback Justification Rate are all computed by slicing the **same tasks** from the benchmarks above by `intervention_layer`. They do not require separate benchmark families.

**Required task-shape metadata in `baseline.json`:**
- `tool_count_class`: `single_tool` | `multi_tool`
- `composition_depth`: `1` | `2_3` | `4_plus`
- `planning_horizon`: `short` | `medium` | `long`
- `cross_api_mashup`: `true` | `false`

**Minimal field schema for `baseline.json`:**

```json
{
  "id": "gaia_001",
  "source": "GAIA",
  "source_split": "level_1",
  "task_family": "research_synthesis",
  "difficulty": "medium",
  "prompt": "the original user request",

  "primary_kpi": [
    "Reasoning-First Success Rate",
    "Decomposition Stability"
  ],

  "tool_count_class": "multi_tool",
  "composition_depth": "2_3",
  "planning_horizon": "medium",
  "cross_api_mashup": true,

  "available_tools_label": [
    "web_search",
    "read_file",
    "write_file"
  ],

  "goal_condition": {
    "type": "text_match",
    "target": "expected answer string",
    "match_mode": "exact"
  },

  "judge_mode": "automatic",
  "risk_tags": [
    "multi_step",
    "tool_choice",
    "context_fragility"
  ],

  "intervention_policy": {
    "reasoning_first": true,
    "soft_scaffold_allowed": ["prompt_scaffold", "memory", "combined"],
    "hard_fallback_allowed": false
  }
}
```

**Intent of the schema:**
- `prompt` is the raw user request passed directly to the observed Agent; do not encode hand-written control flow per task
- `available_tools_label` is diagnostic annotation only, not a routing instruction to the Agent
- `intervention_policy.hard_fallback_allowed` must stay `false` in the mainline experiments and is only reconsidered as a design recommendation in W11
- `primary_kpi` and `risk_tags` are annotations describing what kinds of failures the task is supposed to reveal

**Must confirm in W1:**
- Can each benchmark task actually be run with the OpenClaw / Nanobot tool sets?
- Can we build `baseline.json` after removing impossible tasks?
- Can each task be assigned a `goal_condition`?
- Can each `goal_condition` be reduced to automatic judgment?
- Do we have coverage across `single_tool / multi_tool / cross_api_mashup`?
- Is the distribution of `composition_depth` and `planning_horizon` balanced enough to expose mashup failures in higher-level tasks?

---

## Timeline

### W1 | `00_setup.ipynb` | Setup & Environment

**Goal:** Build a reproducible experimental base and the initial task set

**Required:**
- Verify that OpenClaw / Nanobot start successfully
- Draft `tasks/baseline.json`
- Freeze the `baseline.json` field schema
- Freeze the initial task subset for the reasoning-first baseline
- Declare and export `config.py` from the notebook
- Estimate API cost
- Check benchmark fit
- Define the normalized tool inventory and freeze the tool set and skill-injection conditions shared by OpenClaw / Nanobot
- Decide how to convert each agent's stdout into the shared `log_schema` (stdout parser only vs thin wrapper)

**Nice to have:**
- API budget monitoring
- Check OpenFang logging granularity
- Automate `verify_all()` / `destroy_all()`

**Blog:** "Build a reproducible Agent experiment environment in 5 minutes"

---

### W2 | `01_core_loop_read.ipynb` | Core Loop Anatomy (Part 1)

**Goal:** Read Nanobot and identify exactly what to measure and where

**Read these components:**

```text
agent loop
planner
tool dispatcher
memory manager
retry logic
```

**Tasks:**
- Read the core 500-800 lines with annotated notes inside the notebook
- Visualize measurement points as a structure diagram
- Finalize the log schema
- Separate what can be measured by internal code reading from what must be filled in by external observation

**Blog:** "Reading Nanobot's agent core"

---

### W3 | `02_core_loop_trace.ipynb` | Core Loop Anatomy (Part 2)

**Goal:** Visualize the iterative structure from real logs and finish the KPI aggregation infrastructure

**Tasks:**
- Capture reasoning -> action -> observation -> replanning in logs
- Establish the reference line for `intervention_layer=reasoning_first`
- Visualize structural differences between OpenClaw and Nanobot
- Implement and export `kpi_collector.py`

**Critical path:**

```text
W1: baseline.json / config.py
W2: log_schema JSON Schema frozen
W3: kpi_collector implementation
W4-W11: reused by all later experiments
```

**Degrade order if schedule slips (assuming 10h/week):**

```text
1. Skip W6 Multi-Agent
2. Reduce W5 custom skills from 2 to 1
3. Shorten W10 6h autonomy runs to 3h
4. Fix W9 at N=5 instead of N=10
5. Reduce blog count from target 11 to minimum 8
```

**Degrade decision:**
If `kpi_collector.py` is still unstable by the end of W3, cut W6 Multi-Agent immediately and spend W4-W5 stabilizing the foundation. Do not add any new comparison axis after that point.

**Blog:** "Can an LLM replace if/else?"

---

### W4 | `03_memory.ipynb` | Memory as Soft Scaffold

**Goal:** Measure how much Memory helps when added as a non-code scaffold after the reasoning-first baseline

**Comparison population:**
Use the task subset that still fails under reasoning-first and is frozen by the end of W3. Call this the **scaffold evaluation set**. W4 and W5 both use this same set.

**Tasks:**
- Freeze the scaffold evaluation set
- Run the scaffold evaluation set with `scaffold_type=memory` and compare against reasoning-first in a paired design
- Compare multiple SOUL.md patterns, distinguished by `scaffold_config_id`
- Contrast against memU's knowledge-graph-style approach
- Auto-aggregate KPIs and visualize the deltas

**Blog:** "What does it mean to give memory to an AI?"

---

### W5 | `04_skills.ipynb` | Skill on Top of Memory

**Goal:** Measure the incremental effect of adding Skill as a soft scaffold on top of the W4 Memory scaffold

**Comparison population:**
Use the same scaffold evaluation set as W4. W5 does **not** include a separate skill-only arm. Instead, it treats Skill as a **combined scaffold** added on top of the best memory configuration. The compared task IDs remain aligned with W4.

**Tasks:**
- Analyze ClawHub skills
- Run the scaffold evaluation set with `scaffold_type=combined` and compare it against both reasoning-first and memory-only in a paired design
- Create 1-2 custom skills and compare KPI changes when they are added on top of the best memory configuration; distinguish variants by `scaffold_config_id`
  - Manage these skills as a scaffold skill set; do not modify the baseline tool inventory
  - Do not use skills to inject hand-written control flow
- Observe recovery behavior around ambiguous returns
- Treat this week as measuring the cumulative gain of `memory + skill`, not an isolated skill-only effect

**Blog:** "How do skills change agent behavior and metrics?"

---

### W6 | `05_multi_agent.ipynb` | Multi-Agent as Escalated Scaffold (Optional)

**Primary path:** Deepen the W4-W5 findings and clarify the limit of the single-agent reasoning-first baseline

**Optional path:** Observe multi-agent collaboration as an escalation scaffold

```text
Isolation      -> parallel, independent sessions, separated memory
Collaboration  -> supervisor -> specialists
```

**Gate conditions:**
- Freeze the reasoning-first and soft-scaffolding results first
- If the setup requires new hand-written orchestration code, do not do it
- If the configuration is unstable, prioritize deeper single-agent analysis instead

**Recording rule:**
- If executed, record it as `scaffold_type=multi_agent` and treat it as a separate arm, not as memory / combined
- Store the supervisor pattern and role split in `scaffold_config_id`

**Blog:** "Is an AI team smarter than a single agent?"

---

### W7 | `06_security_failure.ipynb` | Security & Failure

**Goal:** Classify where and why the Agent breaks

**Tasks:**
- Reproduce prompt injection and similar attacks as inputs
- Classify tool-selection failure patterns
- Create a failure taxonomy
- Classify each failure type into `recoverable by reasoning / helped by soft scaffolding / fallback candidate`

**Blog:** "Where does the Agent fail?"

---

### W8 | `07_retry_behavior.ipynb` | Retry Behavior & Self-Repair

**Goal:** Separate LLM self-repair from built-in agent retry and measure how far prompt-level intervention alone can recover failures

**Tasks:**
- Separate `llm_self_repair` / `prompt_scaffold` / `agent_builtin` in logs via `retry_origin`
- Try prompt-level input changes for the top two failure types
- Compare retry success before and after the intervention by `retry_origin`, so prompt effects and built-in retry effects are not mixed
- Do not add any external code-level retry controller

**Blog:** "Are fixed retry rules enough?"

---

### W9 | `08_decomposition.ipynb` | Decomposition Variance

**Goal:** Quantify variance in task decomposition

**Tasks:**
- Run the same request multiple times and collect decomposition logs
- Quantify Decomposition Stability
- Explore the correlation between Plan Length and failure rate

**Blog:** "Measuring variance in task decomposition"

---

### W10 | `09_autonomy_kpi.ipynb` | Autonomy KPI & OpenFang Comparison

**Goal:** Compare across the autonomy spectrum and measure the limit of self-running behavior

**Tasks:**
- Run staged autonomy experiments at 1h / 3h / 6h
- Compare all KPIs across OpenClaw / Nanobot
- When OpenFang is included, compare only the common-KPI subset
- Record Context Degradation over time

**Blog:** "Reactive vs Autonomous"

---

### W11 | `10_synthesis.ipynb` | Final Synthesis

**Goal:** Integrate all findings into a boundary map that NinjaLABO can use for design decisions

#### Research Layer (completed inside the notebook)

- Integrate all experimental results
- Produce the boundary map for LLM control vs code control
- Organize results in three layers: `reasoning-first -> soft scaffold -> fallback candidate`
- Evaluate OpenClaw as a general-purpose PA platform
- Summarize both quantitative findings and qualitative notes about where LLM reasoning breaks and why

**Design decisions finalized in W11:**
- Which task families are good enough with reasoning-first only?
- Which task families are good enough with soft scaffolding?
- Which task families alone justify hard fallback?
- If fallback is justified, is the target a router, a guard, or bounded state handling?

#### Design Layer (written by Hiroshi)

`NinjaLABO_Agent_OS_v0.md`

A document where Hiroshi converts the 11 weeks of internship results into management and design decisions.

```markdown
# NinjaLABO Agent OS v0 - Design Principles

## How does a vertical PA solve problems that a general PA cannot?

| Problem            | General PA (OpenClaw-style)          | Vertical PA (NinjaLABO-style)           |
|-------------------|--------------------------------------|-----------------------------------------|
| Trust             | Structurally hard to solve           | Control blast radius with domain scope  |
| Control flow      | Start with reasoning-first           | Add bounded fallback only afterward     |
| Retry strategy    | General heuristics                   | Adapt using domain-specific failure classes |
| Memory design     | Generic long-term memory             | Domain-specific memory                  |
| Autonomous action | Self-runs on generic tasks           | Self-runs on explicitly defined units of work |

## What we do not implement (YAGNI)

- Re-implement an OpenClaw-compatible framework
- Modify the core of the observed agents
- Add hand-written control flow preemptively
```

**Blog:** "LLM control vs code control: boundary map"

---

## Milestone Review

| Timing | Checkpoint |
|--------|------------|
| **End of W1** | Environment ready, cost estimate captured, baseline draft done, field schema frozen, reasoning-first task subset fixed, normalized tool inventory defined, OpenFang logging granularity checked |
| **End of W2** | Nanobot reading complete, log schema JSON Schema frozen, fallback thresholds provisionally set, 1 article published |
| **End of W3** | KPI automation implemented, reasoning-first reference line frozen, core loop visualization complete, 2 articles published |
| **End of W5** | Memory / Skill soft scaffolding observation complete, 4 articles published |
| **End of W6** | Single-agent deep dive complete (or Multi-Agent observation), 5 articles published |
| **End of W8** | Failure / Retry / Self-Repair observation complete, 7 articles published |
| **End of W10** | Decomposition / Autonomy KPI complete, 9 articles published |
| **End of W11** | All notebooks complete, 8-11 posts published, `NinjaLABO_Agent_OS_v0.md` complete |

---

## Deliverables

| # | Deliverable | Type | Owner |
|---|-------------|------|-------|
| 1 | **Notebook series (11 notebooks)** - reproducible record of experiments, implementation, and analysis | Primary deliverable | Nghi |
| 2 | **Blog posts (8-11)** - auto-generated from notebooks and published | Derived deliverable | Nghi |
| 3 | **Exported Python modules** - log schema, KPI collection, visualization | Derived deliverable | Nghi |
| 4 | **NinjaLABO Agent OS v0.md** - a document translating the boundary map into design decisions | Hiroshi-only deliverable | Hiroshi |

Primary deliverable = notebook.
Derived deliverable = anything mechanically generated from notebooks.
Deliverable #4 is a design document written by Hiroshi using the notebooks as input, so it is outside the notebook-only rule.

---

## Out of Scope

- Modifying the core code of OpenClaw / Nanobot / OpenFang
- Building a new Agent framework
- Inserting hand-written control flow / routers / retry policy before observing the reasoning-first baseline
- Making TypeScript / Rust modification itself the main subject
- Autonomous runs longer than 24 hours
- Production operations or enterprise security work
- Large-scale refactoring of existing code
- Writing hand-made documents outside notebooks
- Mixing model-capacity ablations (for example BitNet) into the mainline comparison
