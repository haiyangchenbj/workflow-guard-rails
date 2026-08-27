---
name: workflow-guard-rails
slug: workflow-guard-rails
displayName: Workflow Guard Rails
description: >
  Wrap multi-step agent workflows with pre-execution checks, side-effect queues,
  result validation, retry budgets, checkpointing, audit logs, and failure-rule
  accumulation. Prevents false successes, duplicate sends, unrecoverable
  crashes, and silent drift in LLM production systems. Use it when a workflow
  sends, publishes, pays, deletes, or writes to another system, runs unattended
  on a schedule, or must be safe to rerun after a mid-task failure. Trigger
  keywords: workflow safety, workflow guardian, agent guard, guardrails,
  pre-execution check, pre-flight check, retry budget, idempotency, false
  success, duplicate send, checkpoint recovery, rerun safety, audit log, drift
  detection, agent reliability, production guardrails, 工作流守护, Agent 护栏,
  副作用队列, 漂移检测, 幂等, 防重复发送, 假成功, 断点恢复, 生产护栏, 重跑安全.
  中文摘要：为多步骤 Agent 工作流加装七项护栏——执行前检查、检查点、副作用队列、预算
  重试、结果验证、审计记录、规则沉淀，拦截假成功、重复发送与渐进漂移。触发词：工作流
  守护、Agent 护栏、假成功拦截、防重复发送、重试预算、断点恢复、生产护栏、漂移检测.
description_zh: 工作流守护护栏：为多步骤 Agent 工作流加装七项护栏（执行前检查、检查点、副作用队列、预算重试、结果验证、审计记录、规则沉淀），拦截假成功、重复发送与渐进漂移。适用于定时运行、无人值守、含发送/发布/写库等外部副作用、需要失败后安全重跑的工作流。触发词：工作流守护、Agent 护栏、假成功、防重复发送、幂等、断点恢复、生产护栏
description_en: A horizontal safety layer for multi-step agent workflows — pre-execution checks, checkpointing, side-effect queues, retry budgets, result validation, audit logs, and rule accumulation.
version: "1.0.3"
agent_created: true
not_for:
  - Single-shot prompts with no external side effects
  - Pure data transforms already covered by unit tests
  - Workflow authoring or design (use a design guide instead)
  - Judgments requiring human taste or policy decisions (use a review skill)
  - Monitoring external systems the workflow does not own
read_when:
  - "workflow safety"
  - "workflow guardian"
  - "agent guard"
  - "guardrails"
  - "pre-execution check"
  - "pre-flight check"
  - "retry budget"
  - "idempotency"
  - "false success"
  - "duplicate send"
  - "checkpoint recovery"
  - "rerun safety"
  - "audit log"
  - "drift detection"
  - "agent reliability"
  - "工作流守护"
  - "副作用队列"
  - "漂移检测"
  - "幂等"
  - "防重复发送"
  - "断点恢复"
  - "生产护栏"
tags:
  - workflow
  - reliability
  - agent-safety
  - guardrails
  - idempotency
  - human-in-the-loop
  - llm-ops
  - automation
  - error-handling
  - checkpoints
  - production-readiness
---

# Workflow Guard Rails

A horizontal safety layer for multi-step agent workflows. It does not replace the workflow logic — it wraps it with guards that catch what the workflow itself cannot see: a tool call that returns "success" but produced invalid output, a side effect that fired before validation, a retry that duplicated an external write, and drift that compounds across runs until the system collapses.

## When to use

- An LLM or agent workflow runs on a schedule or reacts to events, and a failure means broken output, duplicate messages, or an unrecoverable state.
- You have seen "the task reported done but the result was wrong" at least once.
- External actions (send, publish, pay, delete, write to another system) happen as part of the workflow.
- The same workflow runs repeatedly and you want to detect slow drift, not just hard crashes.

## Do not use

- Single-shot prompts with no external side effects — just validate the output directly.
- Pure data transforms where a unit test already covers correctness.
- Cases needing a human to make the judgment call itself (use a review skill instead).

## The seven guards

| # | Guard | What it does | Signal it catches |
|---|---|---|---|
| 1 | Pre-execution check | Verify input completeness, permissions, success criteria, idempotency key | Missing prerequisites, wrong scope |
| 2 | Checkpointing | Split long tasks into recoverable checkpoints | Unrecoverable mid-task crash |
| 3 | Side-effect queue | Hold external actions until validation passes | Broken output reaching users |
| 4 | Retry budget | Rebirth on failure, capped retries, no local patches | Duplicate sends, partial corruption |
| 5 | Result validation | Independent assertions, not "tool returned OK" | False success |
| 6 | Audit log | Record steps, evidence, human confirmations | Lost context, undetectable drift |
| 7 | Rule accumulation | Promote confirmed failures into new guard rules | Recurring, un-codified errors |

> **Detailed patterns**: for per-guard implementation details and example invariants by workflow type, load `references/guardian-patterns.md`.

## Operating procedure

1. **Before execution**: run guard #1. If input, permissions, or success criteria are incomplete, stop and report — do not guess.
2. **During execution**: emit checkpoints (guard #2) at each irreversible boundary.
3. **Before any external action**: push it to the side-effect queue (guard #3). It stays queued until validation passes.
4. **After generation/computation**: run independent assertions (guard #5). Any failure triggers rebirth within the retry budget (guard #4). Local patches are forbidden — rebirth the whole unit.
5. **After success**: write the audit log (guard #6) and propose any new guard rule from confirmed failures (guard #7) for human approval.

## Human in the loop

Require explicit confirmation before:
- First use of a high-privilege tool.
- Any irreversible or external action (send, publish, pay, delete, external write).
- Low-confidence results.
- Retry budget exhausted or validation still failing.
- Writing a new guard rule into the long-term skill.

## Result validation pattern (guard #5)

This is the core. Never treat "the tool call returned successfully" as "the task is done."

```
After the workflow produces output:
  1. Force the model to self-check: output verifiable counts + assertions
  2. Code independently verifies the key invariants
  3. Any assertion fails → rebirth the whole unit (no local patch)
  4. Retry budget reached → report failure, stop, wait for human
```

Key invariants to assert depend on the workflow: field completeness, length limits, format contracts, JSON schema, non-empty critical sections, count equality (e.g. items == summaries).

**Boundary trap**: when asserting `A == B`, also assert `A > 0`. Otherwise `A == B == 0` passes but is clearly broken.

## Side-effect queue pattern (guard #3)

External actions must never fire directly from a "success" signal. They enter a queue and execute only after validation passes. This is what prevents an empty or broken output from reaching users, customers, or other systems.

## Failure-rule accumulation (guard #7)

When a failure mode is confirmed by a human, propose it as a new standing assertion. Recurring issues (e.g. a summary field that keeps exceeding length) should become permanent guards, not one-off fixes.

## Drift monitoring (guard #6)

Do not judge compliance on a single run. Track the compliance rate across runs. A single deviation in one run is an early warning; a trend across runs is an escalation. Catching drift at the first deviation prevents the "looks fine every day, then suddenly collapses" failure path.

## Output format

After each guarded run, emit:

```
🛡️ Guardian Report
- Pre-check: ✅ / ❌
- Checkpoints: N
- Side-effects queued: N, executed: N
- Validation: ✅ / ❌ (assertions: X/Y passed)
- Retries used: N / budget
- Audit log: written / skipped
- New rule proposed: yes/no
- Verdict: ✅ released / ❌ held for human
```

## Limitations

- Guards #5/#6 catch machine-checkable problems (format, count, keyword, length drift). They cannot catch style, semantics, or judgment drift — those need a separate review layer.
- Guards add one validation step per run; budget it as <20% overhead.
- The side-effect queue only protects actions routed through it. Actions fired outside the guardian are not covered.

## Failure Handling

| Scenario | Action |
|---|---|
| Pre-check fails (missing input/permissions) | Stop and report missing items; do not guess |
| Checkpoint crash mid-task | Resume from last checkpoint; do not restart from scratch |
| Validation fails (assertion broken) | Rebirth the whole unit within retry budget; no local patches |
| Retry budget exhausted | Stop, report failure, wait for human confirmation |
| Side-effect queue blocked | Hold all queued actions; do not release until validation passes |
| Audit log write fails | Continue the run but flag the missing audit entry; do not mark fully complete |
| New rule proposed but unconfirmed | Keep as draft; do not activate until human approves |

## Hard Rules

1. Pre-execution check is mandatory; never skip it to save time.
2. External actions must pass through the side-effect queue; no direct fire-on-success.
3. Validation assertions must be independent — never trust "tool returned OK" as success.
4. Rebirth the whole unit on failure; local patches are forbidden.
5. Retry budget is capped; when exhausted, stop and wait for human.
6. Audit log must be written for every run, including failures.
7. New guard rules require human approval before activation.
8. Drift is tracked across runs, not judged on a single run.

---

> **中文导语**：Workflow Guard Rails 是多步骤 Agent 工作流的安全护栏，形态为"固定工作流 + 守护 Skill"，不是自主 Agent。它通过七项守卫（执行前检查、检查点、副作用队列、预算重试、结果验证、审计记录、规则沉淀）拦截"假成功"、重复副作用、不可恢复崩溃和渐进漂移。当你说"这个自动化跑久了会不会哪天悄悄坏了""发出去的内容是不是真的验证过""重试会不会重复发"时，就应该用它。
