# Work Lifecycle

## Contents

- Intake and routing
- Explicit Work and Request links
- Handoffs
- Tasks and execution
- Watching and follow-ups
- Pull request delivery
- Completion and review

## Intake and Routing

Treat the user's message as intake, then choose the durable execution context.

1. Extract any Vector URL or key. Work URLs normally end in a key such as
   `AUTH-42`; Request keys commonly use a Request-specific prefix.
2. Resolve explicit Work with `vcli work context <key>`. Resolve explicit
   Requests with `vcli request get <key>` and inspect linked Work.
3. Without an explicit link, query owned and active Work, then search using the
   outcome, feature, project, team, repository, and distinctive nouns.
4. Rank candidates by semantic outcome match first. Use project/team,
   ownership, active status, linked Request, existing branch/PR, and recent
   activity as supporting evidence.
5. Reuse only a clearly matching Work item. If candidates are ambiguous and
   choosing incorrectly would change ownership or scope, ask one concise
   question.
6. For genuinely new work, create the Request first so the user's expected
   output remains reviewable. Create Work linked to that Request.

Use `fulfills` when Work is responsible for delivering the Request's expected
output. Use `contributes` when Work supplies context or a partial contribution
without controlling Request completion.

## Explicit Work and Request Links

Interpret a Work link plus an action request as authorization to use that Work
as context and explicitly start or resume it. Inspect a bare link before acting;
default to continuation when the conversational context is an execution handoff
or follow-up, and default to a read-only summary when it is clearly referential.

Always load the bounded Work context rather than reconstructing it through
separate entity queries. Re-read it whenever the user changes requirements.

For a linked Request, inspect its expected output, review guidance, recipients,
and linked Work. Attach it to matching Work or create Work; do not treat Request
acceptance as Work start.

## Handoffs

Identify the current user with `vcli auth whoami`. In Work context, inspect:

- pending and accepted handoff IDs and timestamps;
- outgoing and incoming owners;
- summary and note;
- referenced Task or stated resume point;
- ownership history and current Work status;
- open Tasks, blockers, attention requests, executions, branches, and PRs.

When a pending handoff targets the current user, a message such as “continue
this,” “take this over,” “work on this,” or a Work link supplied as the task is
sufficient intent to accept. If the user is merely asking what the link is,
summarize the handoff without accepting it.

After acceptance, explicitly start Work. Continue from the latest valid resume
point and preserve previous progress. If the prior execution is resumable,
resume it; otherwise start a new execution attached to the same Work.

## Tasks and Execution

Use Tasks for boundaries that benefit from durable state. Typical examples:

- schema or migration work before dependent code;
- backend contract and frontend integration;
- independent repositories or PRs;
- a verification/review pass with separate acceptance criteria;
- an external decision, blocker, or human handoff;
- parallel work assigned to different people or agents.

Keep short implementation notes in the workpad. Avoid Tasks that only restate
individual coding steps.

When an agent has a live activity ID, pass it through `--execution` on Task
creation and attention requests. This records provenance and enforces the Work
agent policy. Without a real ID, omit the option.

## Watching and Follow-ups

Start monitoring after Work selection, not after implementation. The baseline
watch categories are `work,tasks,requests,attention,handoffs,executions`.

Use a continuous watcher when the current environment can keep and observe a
background process. Use a heartbeat/recurring automation when the environment
can wake the task later. A bounded watcher is useful for one dependency:

```bash
vcli --json work watch AUTH-42 \
  --events tasks,requests,attention,handoffs,executions \
  --once \
  --timeout 1800
```

Each JSON event is one NDJSON line. Use `--initial` only when the initial full
snapshot is also needed. On an event:

1. Ignore duplicates already processed by the current execution.
2. Refresh `work context`.
3. Determine whether the change adds scope, changes an acceptance criterion,
   resolves a blocker, requests a handoff, or supplies review feedback.
4. Update Tasks and the current implementation/PR as needed.
5. Restore an accurate Work state and continue watching.

Keep monitoring through `ready-for-review` and through the agreed follow-up
window after completion. A user change request, new linked Request, or new Task
resumes the same Work. If Work was already marked ready or complete, restore an
active state as supported, move the relevant Task back to `in_progress`, make
the change on the same open PR, re-run checks, and mark ready again.

Monitoring is a real capability claim. Never say Work is being watched after
the watcher/automation has stopped.

## Pull Request Delivery

Inspect GitHub evidence before creating anything. Prefer the existing open PR
whose repository, branch, and scope match Work.

For a new PR:

1. Use a narrow branch and the repository's expected base.
2. Include the Work key/URL in the PR body and describe linked Request expected
   outputs.
3. Link the PR back to Work with the compatibility command:

```bash
vcli issue link-github AUTH-42 https://github.com/acme/app/pull/123
```

4. Verify the PR appears in the next `work context` snapshot.

For follow-ups, commit and push to the same branch while the PR is open. If it
is merged, create a follow-up PR linked to the same Work when the change still
belongs to that outcome; create separate Work only for a genuinely different
outcome.

## Completion and Review

Use `vcli work ready-for-review <key>` when delivery evidence and checks are
ready for the requester. Keep the watcher active for change requests.

Use `vcli work complete <key>` only when the Work completion policy is met.
Request completion is a separate requester-review action. Do not conflate an
agent finishing a turn, a PR being opened, Work being ready for review, and the
requester accepting the outcome.
