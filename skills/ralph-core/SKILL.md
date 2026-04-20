---
name: ralph-core
description: Manual-only persistence execution protocol with a mandatory implementation-review loop. Use this skill only when the user explicitly invokes `$ralph-core` or explicitly asks for `ralph-core`. Do not auto-apply.
---

# Ralph Core

## Purpose

Use Ralph Core as a persistence workflow for execution tasks that must be completed with fresh verification evidence and a mandatory independent review loop.

Treat it as protocol only. Keep advancing the task, keep verifying, hand each implementation wave to another subagent for review, and do not declare completion without evidence and review approval.

## Manual Trigger Only

Use this skill only when the user explicitly says one of the following:

- `$ralph-core`
- `ralph-core`
- `use ralph-core`
- `run ralph-core`

## What This Skill Does

Use Ralph Core to enforce this loop:

1. Ground the task before heavy execution
2. Convert the requested contents into an explicit completion ledger
3. Keep working through bounded next steps
4. Verify the current implementation wave with fresh evidence
5. Hand the implementation wave to another subagent for review
6. Fix failures, omissions, and review findings, then re-verify and re-review
7. Finish only when completion is supported by evidence and the independent review passes

Use it to prevent shallow fixes, partial delivery, false completion, and unverified "done" claims.

## Use When

Use Ralph Core after explicit manual invocation when the task benefits from persistence pressure:

- multi-step implementation that may require retries
- debugging that needs repeated fix/verify cycles
- tasks where "done" must be proven, not guessed
- work with multiple independent subtasks that can be parallelized
- follow-through tasks where partial completion would be misleading
- tasks that require implementation plus independent review before completion

## Core Principles

- Do not declare completion without fresh verification evidence.
- Do not reduce scope unless the user explicitly changes scope.
- Prefer the smallest meaningful next step that advances the task.
- Run independent work in parallel when the host supports it.
- Read verification output; do not infer success from confidence alone.
- If verification fails, fix the concrete issue and rerun verification.
- After every implementation wave, obtain review from another subagent before considering that wave complete.
- Treat review findings as blocking until fixed or explicitly disproved with evidence.
- Do not complete while any planned item, accepted scope item, failing verification, or unaddressed review finding remains open.
- Keep progress concise and evidence-dense.
- Preserve reversibility when the next step is low-risk and clear.
- Escalate only when blocked by ambiguity, missing access, destructive choice, or external dependency failure.

## Lightweight Local Artifacts

When local artifacts are useful, maintain a lightweight working set such as:

- task snapshot
- completion ledger
- progress ledger
- changed-files list
- latest verification evidence
- latest review findings

A default directory such as `.ralph-core/` is acceptable, but this skill does not require any specific runtime-owned path.

Recommended artifact examples:

- `.ralph-core/task.md`
- `.ralph-core/completion-ledger.md`
- `.ralph-core/progress.md`
- `.ralph-core/changed-files.txt`
- `.ralph-core/verification.md`
- `.ralph-core/review.md`

If local artifact storage is unnecessary, keep the same structure conceptually in working memory and progress updates.

## Execution Flow

### 1. Ground the task

Establish the minimum reliable context before heavy execution:

- task statement
- desired outcome
- explicit requested contents and deliverables
- known facts or evidence
- constraints
- unknowns or open questions
- likely touchpoints
- likely verification commands

Clarify first when the task is too ambiguous to verify or scope safely.

### 2. Create a completion ledger

Translate the request into an explicit ledger of what must be true before completion.

Capture at minimum:

- required deliverables
- acceptance criteria
- known planned items
- open questions that block implementation
- items already complete
- items still pending

Use this ledger as the source of truth for "all content has been fully implemented". Do not rely on vague memory of the request.

### 3. Resume if prior work exists

Continue from prior work instead of restarting when reliable artifacts or evidence already exist.

Reuse:

- confirmed facts
- completion ledger state
- completed subtasks
- identified changed files
- last known failing verification result
- last known review findings

Do not duplicate solved work unless there is reason to distrust it.

### 4. Pick the next bounded slice

Choose the highest-leverage incomplete step that can be executed now.

Prefer work that is:

- concrete
- testable
- low-risk
- locally reversible
- likely to unblock later verification

Avoid broad rewrites when a smaller verified step can move the task forward.

### 5. Parallelize independent work when appropriate

Use subagents, workers, or parallel tool execution when the host supports them and the tasks are independent.

Good candidates:

- read-only codebase exploration
- separate file family analysis
- isolated implementation tasks with non-overlapping write scope
- independent regression checks

Do not parallelize tightly coupled blocking work when the immediate next step depends on one result.

### 6. Record changed files

Maintain a bounded list of changed files during the session.

Use the list to:

- keep task scope visible
- focus later cleanup or review
- support final reporting
- prevent unrelated edits from expanding the session boundary

### 7. Verify the implementation wave with fresh evidence

After each implementation wave, run fresh verification before review.

An implementation wave means any round that changed code, prompts, files, scripts, configuration, or other task artifacts.

Identify what proves the current wave is correct before review and what proves the whole task is done before completion.

Typical evidence:

- tests pass
- build succeeds
- lint or typecheck succeeds
- runtime behavior is reproduced or fixed
- required output is generated
- the relevant regression no longer occurs

Verification requirements:

- run the relevant command now
- inspect the result now
- confirm it matches the desired outcome
- do not rely on stale runs
- do not rely on intuition

### 8. Run a mandatory independent review

After each implementation wave, hand the changed work to another subagent for review.

Review requirements:

- the reviewer must be another subagent, not the same worker that made the change
- the review must check both completeness and correctness
- the review must explicitly look for missing planned content, regressions, and latent problems
- the review result must be treated as pass or fail, not casual commentary

Minimum reviewer questions:

- Were all planned or requested contents for this wave actually implemented?
- Is there any missing work relative to the completion ledger?
- Are there correctness, regression, reliability, maintainability, or edge-case issues?
- Should the current wave be accepted, or must implementation continue?

If the host cannot provide another subagent for independent review, do not claim the full Ralph Core contract was satisfied. Report that the required review loop is unavailable.

### 9. Fix, re-verify, and re-review when needed

When verification fails, or when the reviewer does not pass the wave:

1. identify the specific failure
2. update the completion ledger and open-items list
3. fix the concrete issue or missing content
4. rerun the relevant verification
5. send the updated wave back to another subagent for review
6. repeat until verification is green and review passes, or until a real blocker is explicitly evidenced

Do not stop at "likely fixed", "mostly done", or "should work now".

Do not label work as blocked merely because the fix is hard, repetitive, time-consuming, or uncertain. A blocker must identify the exact external dependency, missing approval, missing information, unavailable capability, or safety constraint that prevents the next local implementation step.

### 10. Use stronger review when risk is high

Escalate the reviewer quality when the change is broad, risky, or high-impact.

Examples:

- architectural review
- correctness review
- security review
- regression-focused review

This adjusts reviewer depth, but does not remove the mandatory review loop.

### 11. Complete only when all implementation and review conditions are satisfied

Require all of the following before completion:

- every requested or planned content item in the completion ledger is complete, or any remaining gap is explicitly accepted by the user
- the requested work is actually implemented or otherwise completed
- current blocking issues are resolved, or remaining blockers are explicit
- fresh verification evidence exists
- the latest independent subagent review passes
- there are no unaddressed review findings
- no critical unresolved failure is being hidden behind optimism
- residual risks, if any, are clearly stated

## Verification Discipline

Use the strongest practical verification for the task.

Preferred order:

1. direct regression reproduction and fix confirmation
2. automated tests
3. build, typecheck, or lint
4. targeted runtime checks
5. artifact or output inspection

If strong verification is unavailable, say so explicitly and use the best available evidence. Never quietly downgrade verification strength.

## Mandatory Implementation-Review Loop

The Ralph Core loop is not only implement-then-verify. It is implement, verify, review, fix, then repeat.

Required loop:

1. implement the next bounded slice
2. run fresh verification for that wave
3. send the wave to another subagent for review
4. if review fails or finds missing content, continue implementation
5. rerun verification
6. send the updated wave for review again
7. repeat until the wave passes and the completion ledger has no unresolved required items

Never treat a self-check as a substitute for the required independent subagent review.

## Optional Companions

Use companion workflows only when they are available and helpful.

Examples:

- clarification or interview workflow before execution
- planning workflow before execution
- visual QA workflow for screenshot tasks
- cleanup or deslop workflow on changed files only
- independent reviewer or architect workflow
- explicit cancel or cleanup helper

Treat all companions as optional host capabilities, not core requirements.

## Escalation and Stop Conditions

Stop and report when:

- the user withdraws or redirects the task
- required credentials, permissions, or external systems are unavailable
- the required independent subagent review capability is unavailable
- the next step is materially destructive and not yet approved
- the task is too ambiguous to execute safely
- repeated cycles expose a fundamental blocker rather than a fixable implementation issue

When the same root cause repeats, surface it explicitly as a likely blocker instead of pretending more retries alone will solve it.

When reporting a blocker, include all of the following:

- the exact blocked item from the completion ledger
- the reason the next local implementation step cannot safely proceed
- the evidence for that blocker
- what user input, approval, access, or external recovery is required to continue

Do not present a task as complete when it is only blocked.

## Reporting Style

Report progress and results in a style that is:

- concise
- evidence-dense
- grounded in actual execution
- explicit about what passed, what failed, and what remains

Prefer:

- conclusion
- evidence
- impact
- residual risk

Avoid:

- motivational filler
- vague confidence language
- "should work" completion claims without evidence

## Final Checklist

- [ ] The user explicitly invoked `ralph-core`
- [ ] Task grounding exists
- [ ] A completion ledger exists
- [ ] The next-step sequence stayed bounded and concrete
- [ ] Changed files were tracked
- [ ] Fresh verification evidence exists
- [ ] Each implementation wave was handed to another subagent for review
- [ ] Failed verification or failed review was followed by fix-and-reverify-and-rereview
- [ ] All required completion-ledger items are complete
- [ ] The latest independent review passed
- [ ] No review findings remain open
- [ ] No premature completion claim was made
- [ ] Any residual risk or blocker is explicitly stated
