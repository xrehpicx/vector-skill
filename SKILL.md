---
name: task-management
description: Manage Vector Requests, Work, and Tasks with vcli. Use whenever a user gives Codex a work request, a Vector Work/Request URL or key, an incoming handoff, a follow-up to existing work, or asks for code delivery that should stay linked to Work and a pull request. Also use for Vector CLI setup, workspace operations, live Work watching, agent execution attribution, requester review, and legacy Issue compatibility.
---

# Vector Task Management

Treat Vector as the durable source of truth for execution. Use the CLI to keep
the user's request, active Work, Tasks, agent execution, handoffs, and delivery
evidence connected throughout the job.

The primary model is:

- **Request** captures intake, expected output, routing, and requester review.
- **Work** is the durable execution context. It contains the workpad, linked
  Requests, Tasks, ownership periods, handoffs, agent executions, and GitHub
  evidence.
- **Task** is an optional tracked step within Work. Create Tasks only when a
  step benefits from its own status, assignee, blocker, handoff, or execution
  attribution.

## Route Every Work Request

For every request that may require tracked work:

1. Detect a Vector workspace/organization, Request, or Work key or URL in the message.
2. Resolve the workspace before reading or mutating data. Prefer, in order: the
   org slug in a Vector URL, an explicit user choice, the workspace of referenced
   Request/Work, or the only available membership. For a bare key with no slug,
   run `vcli org list` and probe the key read-only with explicit `--org` values;
   proceed only when exactly one workspace matches. If new work could belong to
   multiple workspaces, ask instead of using the profile's last active org.
3. Store the resolved slug as `ORG` and pass `vcli --org "$ORG"` on every
   Request, Work, Task, watcher, attachment, search, and metadata command for
   this job. Do not run `org use` merely to change context for one job.
4. If Work is explicit, run `vcli --org "$ORG" work context <key>` before planning or acting.
5. If no Work is explicit, inspect likely existing Work before creating any:
   use `vcli --org "$ORG" work list --scope mine`, `vcli --org "$ORG" work list --scope active`, and
   focused `vcli --org "$ORG" search` queries.
6. Reuse Work when the outcome, project/team, ownership, repository evidence,
   and current activity show it is the same delivery context.
7. Otherwise create a Request with a concrete expected output using
   `vcli --org "$ORG" request create`, then create new Work in the same explicit
   workspace and link it to that Request.
8. Keep new follow-up instructions attached to the chosen Work. Link a new
   Request when the instruction introduces a separately reviewable expected
   output; use a Task or workpad update for an execution detail within the
   existing outcome.

Do not create duplicate Work merely because the wording differs. Do not attach
unrelated requests to convenient active Work.

## Interpret Work Links as Intent

A user who supplies a Work URL/key in an execution request is normally asking
the agent to start or continue in that Work context. A bare Work link should be
inspected first and treated as continuation intent unless the surrounding
conversation clearly makes it informational.

After resolving the key:

1. Run `vcli auth whoami`, resolve `ORG` as described above, and run
   `vcli --org "$ORG" work context <key>`.
2. Read linked Requests and expected outputs, Tasks, workpad, blockers,
   attention items, ownership periods, handoffs, agent executions, recent
   activity, and GitHub evidence.
3. Reconcile the user's newest instruction with that context. The newest
   explicit instruction wins where it intentionally changes scope; preserve
   unrelated accepted requirements.
4. Explicitly start/resume Work with `vcli --org "$ORG" work start <key>` when the link and
   request communicate execution intent. Assignment or attachment alone is not
   enough.
5. When execution begins inside a supported local agent, attach the current
   agent session with `vcli --org "$ORG" --json work attach-session <key>`. Keep the returned
   real `liveActivityId` as the execution ID for Task and attention commands.
   The command is idempotent for the same active session; never invent an ID.
   If attachment reports that the bridge is not configured or not running, run
   `vcli service start`, verify it with `vcli service status`, and retry the
   attachment before continuing.

## Resume Incoming Handoffs

When Work was handed off to the current user:

1. Compare `vcli auth whoami` with the incoming owner in `work context`.
2. Inspect the latest handoff status, summary, note, referenced Task, completed
   work, open blockers, attached execution, branch, and PR evidence.
3. If the handoff is pending and the user's request/link clearly asks to take
   over or continue, accept it with `vcli --org "$ORG" work respond-handoff <id> --accept true`.
   If intent is unclear, explain the pending handoff and ask before accepting.
4. Run `vcli --org "$ORG" work start <key>` after acceptance. Accepting a handoff does not
   start the new ownership period.
5. Resume at the stated Task or next unfinished boundary; do not restart the
   Work or discard the prior owner's context.

## Plan With Meaningful Tasks

Create Tasks before implementation when the Work has independently trackable
units such as a backend contract, UI surface, migration, test/verification
pass, external dependency, blocker, parallel owner, or handoff boundary.

- Do not create one Task per trivial checklist item.
- Reuse existing unfinished Tasks before adding duplicates.
- Set a Task `in_progress` when acting on it and `done` only after its acceptance
  check passes.
- Use `waiting` or `blocked` when progress genuinely depends on another actor
  or condition.
- Pass the real `liveActivityId` through `--execution` when available. Never
  invent an execution ID.

## Keep Work as Live Context

Refresh `vcli --org "$ORG" work context <key>` at meaningful boundaries: before planning,
before starting a Task, after a watcher event, before publishing delivery, and
before responding to a follow-up.

Start a Work watcher as soon as Work becomes the active context. Watch at least
`work,tasks,requests,attention,handoffs,executions`; narrow categories only when
the wait condition is deliberately smaller.

- Prefer a native background/heartbeat automation when the agent environment
  supports one.
- Otherwise keep `vcli --org "$ORG" --json work watch <key>` running in a background terminal
  that the harness can observe.
- For scheduled wakeups, use `--once --timeout <seconds>` and relaunch through
  the environment's recurring mechanism instead of shell polling.
- Deduplicate events, refresh context, and act only on newly observed changes.
- Continue watching while Work is active, ready for review, or in the agreed
  follow-up window after completion. Do not stop merely because the first
  implementation turn, PR, review, or completion transition is finished.
- On a follow-up, resume the existing agent session and Work/Task/branch/PR when
  available rather than creating a parallel context.

If the environment cannot maintain a background watcher after the current
session ends, say so and leave the Work in an accurate state instead of
claiming it is still monitored.

The local Vector bridge must be running for session attachment and inbound
messages. An attachment failure caused by an unconfigured or stopped bridge is
an instruction to run `vcli service start`, confirm `vcli service status`, and
retry—not a reason to omit the Work Session. On macOS the service then starts at
login and restarts after failures. Each agent session attaches independently,
so multiple agents can appear on the same Work and receive messages from the
web or iOS Work page.

## Deliver Code Through One Linked PR

For code-related Work:

1. Inspect GitHub evidence in `work context` before creating a branch or PR.
2. Reuse the existing open PR and branch when they represent this Work.
3. Otherwise create a focused PR and link it to Work with
   `vcli --org "$ORG" issue link-github <workKey> <prUrl>` (legacy command name; Work uses the
   same underlying record).
4. Mention the Work key/URL in the PR title or body when repository conventions
   allow it.
5. Keep follow-up commits on the same PR while it remains the delivery vehicle.
   Create another PR only when the earlier PR is merged/closed or the new change
   is independently reviewable.
6. Mark Work ready for review only after the expected output is represented by
   linked delivery evidence and relevant checks pass.
7. Keep watching for requester changes, new linked Requests, Tasks, handoffs,
   and review feedback; update the same PR and return Work to review.
8. After completion, route a same-outcome follow-up back into this Work and its
   existing PR context. Reopen/resume Work as supported; create new Work only
   when the follow-up is a genuinely separate outcome.

Never merge unless the user explicitly authorizes it. Do not mark the Request
complete on the requester's behalf unless that action is clearly authorized.

## Read References as Needed

- Read [references/work-lifecycle.md](references/work-lifecycle.md) for the
  detailed routing, handoff, watcher, follow-up, and PR lifecycle.
- Read [references/work-commands.md](references/work-commands.md) for exact
  Request, Work, Task, review, and legacy Issue commands.
- Read [references/cli-operations.md](references/cli-operations.md) for install,
  authentication, organizations, teams, projects, documents, RBAC,
  notifications, discovery, scripting, and troubleshooting.
- Read [references/agent-bridge.md](references/agent-bridge.md) when setting up
  or diagnosing local agent sessions, execution attribution, or the Vector
  bridge service.

Use `--json` for scripting. For agent-managed Request/Work, always use explicit
`--profile` when profiles are ambiguous and explicit `--org` after resolving the
workspace; never treat the last active org as evidence. Run `vcli --org "$ORG" refdata`
before guessing workspace keys or metadata.
