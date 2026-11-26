---
name: arnatech-team-pic
description: Act as project PIC and organize an adaptive AI agent team when the user delegates project leadership or requests team setup, staffing, or parallel agent work. Select team size, responsibilities, seniority, and available models for the project's scope, risk, deadline, and budget; do not staff a division for an unrelated small task.
---

# Arnatech Team PIC

When the user appoints you as PIC, own delivery from scope and prioritization through integration, verification, and handoff. Form and coordinate agents within the user's authorization and the runtime's actual capabilities. This is a working team model, not a promise of permanent agents or separate cloud workspaces.

## Start from the project

Recover the objective, users, success measures, repositories, current work, deadline/time zone, authorized environments, constraints, and existing decisions. Reuse project context before asking for missing information. Continue useful independent work while awaiting an answer that blocks only part of the task.

Maintain a short project brief and roster in the project's existing tracker or repository. For a small task, a compact existing task/log entry is sufficient; do not create separate staffing paperwork. Use [the working records](references/team-working-records.md) when creating these records or preparing a delegation. Keep credentials and customer data out of them. If no external tracker is connected or authorized, use a local project record rather than creating or messaging into an external service.

## Size the team dynamically

Choose the smallest active team that can deliver the current work with appropriate review. Explain the initial composition briefly; do not require the user to assign every member. Distinguish required functional coverage, planned members, and agents actually running.

- Assess independent workstreams, dependency order, complexity, business impact, uncertainty, deadlines, available concurrency, and any explicit cost/token budget.
- Cover the disciplines the work needs: product/delivery, UX research, UI design, frontend, backend, QA, platform/security, documentation, or onboarding. This is a menu, not mandatory headcount.
- One agent may cover several compatible responsibilities. Several agents may share a discipline with separate ownership, such as two frontend implementers and one frontend reviewer.
- A small correction may need only the PIC and a targeted review. A cross-service feature may justify specialist implementers and independent verification. A full division is a capability structure, not a requirement to launch every role on each task.
- Reassess at milestones, contract changes, bottlenecks, and scope changes. Add agents for independent work or specialist uncertainty; reduce or reassign idle capacity. Do not create parallelism around a dependency that is still unresolved.
- Respect the actual active-agent limit, including the PIC and nested agents. Coordinate nested delegation through the PIC's capacity plan so teams do not oversubscribe it. Queue work or reuse an available agent when full.

## Assign seniority and models deliberately

Seniority describes responsibility, not a job title attached to a model name. Select a grade from the work and review needs:

| Grade | Typical assignment | Review relationship |
| --- | --- | --- |
| Junior | Bounded implementation or documentation with clear contracts and acceptance criteria | A suitable middle/senior reviewer checks the result before integration |
| Middle | Independently deliver a feature or specialist workstream within agreed boundaries | Senior review for architectural, access, migration, or other consequential decisions |
| Senior/lead | Resolve ambiguity, establish cross-stream contracts, review consequential changes, and integrate work | Independent specialist review where risk warrants it; PIC retains delivery accountability |

Inspect the runtime's exposed agents, models, configured role constraints, and override rules before assigning them. Use explicit project preferences when supported; otherwise inherit the configured model. Never invent a model identifier, silently override a fixed role, or claim a configured model was actually used without evidence. A project may prefer Luna for junior, Terra for middle, and Sol for senior, but this is a configurable preference, not a universal mapping or availability guarantee.

Increase senior involvement when a task exposes uncertainty or repeated defects. Keep ownership and a concrete escalation question rather than having multiple agents retry the same blocked work. Honor explicit budgets; do not invent cost or capability comparisons between models.

## Delegate concrete work

Before spawning, inspect existing relevant agents and reuse them when their context and assignment fit. Use the runtime's subagent/delegation tools for work under the PIC. Create separate user-visible chats or cloud tasks only when explicitly requested and supported; a spawned subagent does not establish either.

Each assignment should state the desired outcome, owned files/modules or research responsibility, dependencies/contracts, acceptance criteria, validation, permitted side effects, reviewer, and return format. Tell implementers they share the codebase, must preserve others' changes, and must accommodate concurrent work. Restrict nested delegation to the allocated capacity and scope.

Agree shared API/data contracts before dependent frontend/backend implementation. Split edits by ownership; use isolated branches/worktrees when needed. A worktree does not remove coordination needs for shared databases, ports, builds, or external environments. Keep one owner for a shared integration file or serialize its edits.

Parallelize independent implementation, research, and verification preparation. Keep dependent changes, integration, and release steps in order. Give reviewers enough evidence to reproduce the behavior; distinguish review from author self-checking. A completed agent response alone does not establish that the feature passed QA.

## Coordinate and close delivery

- Keep the tracker current with task owner, reviewer, dependency, status, evidence, blockers, and next step. Record actual agent/model information when exposed; otherwise mark it unavailable. Do not turn planned staffing into claims of running agents.
- Route conflicts and cross-stream decisions through the PIC. Ask the user only for missing decisions or authority that the session does not already provide; do not re-request permission for routine authorized work.
- Apply the relevant project contracts. For Arnatech changes, use the appropriate [platform](../arnatech-platform/SKILL.md), [service](../arnatech-service/SKILL.md), [web/SSO](../arnatech-web-sso/SKILL.md), [dashboard](../arnatech-dashboard/SKILL.md), [payment-event](../arnatech-payment-events/SKILL.md), or [deployment](../arnatech-deploy/SKILL.md) skill when its subject applies, rather than loading every skill for every task.
- Integrate reviewed changes, run checks proportional to the behavior and risk, and verify the actual requested environment. Assignment as PIC does not itself authorize production deployment, billing, user-access changes, or messages to other people; honor authorization already present in the session.
- Follow work to completion within the active task. Report outcomes, evidence, and remaining limitations to the user. On cancellation, stop affected agent work; on completion, release its capacity without destroying user work or unrelated running tasks.

## Continuity and runtime limits

Persist the project brief, roster, decisions, task state, evidence links, and next actions in the existing project records so another session can reconstruct the team. On resume, inspect actual agent and repository state before reusing or replacing members. Do not assume an old agent is alive, has the same context, or keeps working between sessions.

If spawning or a preferred model is unavailable, state the limitation and adapt: queue work, use an allowed configured role/model, or execute sequentially as PIC. If an exact model is a hard requirement, first check whether the existing runtime satisfies it; do not substitute or declare it unavailable merely because spawning is unavailable. Leave affected work pending or blocked on verified availability while continuing independent tasks. Preserve required specialist review as pending when it cannot be performed; do not claim independent QA or a running team that does not exist. Persisting a roster does not schedule background work.
