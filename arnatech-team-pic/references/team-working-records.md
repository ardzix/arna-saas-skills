# Team working records

Use the project's existing tracker structure. The fields below are prompts for information to retain, not a requirement to create duplicate documents or a new management system. Short tasks can keep a compact combined record.

## Project brief and roster

Record the objective and success measure, in/out of scope, repositories, time zone/deadline, relevant contracts, authorized environments/actions, explicit budget, and unresolved decisions. List the current milestone and its acceptance criteria.

| Roster field | Purpose |
| --- | --- |
| Responsibility and grade | Why the role is needed and what decisions it owns |
| Owner/agent identifier | Actual running or reusable agent; leave unset for a planned role |
| Model | Runtime-confirmed configured/actual model when exposed, or unavailable; keep preferred model separate |
| Owned area | Files, modules, workstream, or research question |
| Reviewer | Person/agent responsible for acceptance review, or review pending |
| State | Planned, running, waiting on dependency, reviewing, complete, or unavailable |
| Dependencies and evidence | Input contracts, task/PR links, checks, blockers, and next action |

Distinguish the project's long-term capability roster from its active execution team. Record why composition changed, especially when a deadline, budget, risk, or runtime limit changed the allocation.

## Delegation brief

Supply only the context needed for this assignment. A practical message can take this form:

```text
Outcome: [specific deliverable tied to a project task]
Ownership: [files/modules or bounded research responsibility]
Inputs and dependencies: [agreed contracts, relevant skills, source/evidence]
Acceptance and validation: [observable behavior and appropriate checks]
Execution boundary: [read-only/implementation; allowed environments and actions]
Review and handoff: [reviewer; summary, changed files/commit, evidence, limits]
Capacity: [nested delegation allocation, or none]

You are not alone in the codebase. Preserve others' changes and adapt to concurrent
work. Coordinate shared-file changes with the PIC before editing them.
```

These brackets are fields for a real assignment, not claims that a task has been assigned. Do not send an unfilled template to an agent.

## Examples of adaptive composition

**Small UI correction.** PIC implements or assigns a bounded frontend change. Use an available reviewer for the affected behavior when needed. One person/agent can cover frontend and UI; a separate backend, security, and UX research team is unnecessary unless the change affects those concerns.

**Feature across frontend and backend, four active slots including PIC.** A possible initial allocation is PIC, frontend implementer, backend implementer, and a QA/reviewer. The QA/reviewer can prepare checks while implementation proceeds; agreement on API and permission contracts precedes dependent edits. If specialist UX work must finish first, temporarily allocate a slot to UX and stage the implementation or review roles afterward. The four-slot example is not a default limit or mandatory team size.

**Several projects or a larger product division.** Maintain capability coverage across product, design/research, engineering, quality, platform, and enablement as needed. Activate members around current workstreams and actual runtime/budget limits. A frontend lead may review multiple bounded frontend assignments while a backend lead coordinates service contracts. Do not copy BNK headcount, model assignments, or business priorities into another project without assessing its needs.

**Model preference unavailable.** Keep the requested grade and reviewer relationship, but disclose the unsupported model and choose an allowed alternative when the preference is not a hard requirement. If the exact model is a hard requirement, queue that work as blocked on availability and continue independent work. An inherited or fixed-role model must not be reported under the preferred model's name.

## Status and handoff

Update existing task/log/QA records with the actual result, evidence, limits, and decision. A useful handoff identifies what changed, who owns follow-up, what passed, what failed or remains unverified, the integrated revision, and the next action. Keep synthetic checks, production probes, authenticated browser verification, and user acceptance distinct.

For cross-session continuity, retain the roster and unresolved work in a durable project record. Reconcile that record with live tools and repositories on resume; it is not evidence that agents continued running while the session was inactive.
