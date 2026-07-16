# Request, Work, and Task Commands

## Contents

- Requests
- Work
- Watching Work
- Handoffs and attention
- Tasks
- GitHub evidence
- Legacy Issues

## Requests

Resolve the workspace first and keep it explicit. The examples below assume
`ORG=acme`; do not substitute the profile's active org when multiple workspaces
are plausible.

```bash
vcli --org "$ORG" request list --scope inbox
vcli --org "$ORG" request list --scope mine
vcli --org "$ORG" request list --scope requested
vcli --org "$ORG" request get REQ-12

vcli --org "$ORG" request create \
  --title "Add enterprise SSO" \
  --description "Customer needs SAML for the admin console" \
  --expected-output "Admins can configure SAML and users can sign in" \
  --review-guidance "Validate with customer metadata" \
  --recipients "alice@example.com,bob@example.com"

vcli --org "$ORG" request route REQ-12 "alice@example.com,bob@example.com"
vcli --org "$ORG" request claim REQ-12
vcli --org "$ORG" request link-work REQ-12 AUTH-42 --relation fulfills
vcli --org "$ORG" request link-work REQ-12 DOCS-7 --relation contributes
vcli --org "$ORG" request request-changes REQ-12 --note "SCIM is still missing"
vcli --org "$ORG" request complete REQ-12 --note "Validated with the requester"
```

Every Request requires `--expected-output`. Routing, claiming, or linking a
Request never starts Work.

## Work

```bash
vcli --org "$ORG" work list --scope active
vcli --org "$ORG" work list --scope mine
vcli --org "$ORG" work list --scope attention
vcli --org "$ORG" work get AUTH-42
vcli --org "$ORG" work context AUTH-42
vcli --org "$ORG" work context AUTH-42 --task 3

# Attach the current Codex/Claude session to Work
vcli --org "$ORG" work attach-session AUTH-42
vcli --org "$ORG" --json work attach-session AUTH-42 --task 3

# Use an explicit detected session only when auto-detection is ambiguous
vcli --org "$ORG" work attach-session AUTH-42 --session <sessionKey>

vcli --org "$ORG" work create \
  --title "Enterprise identity rollout" \
  --workpad "Notes, decisions, and live checklist" \
  --owner alice@example.com \
  --request "REQ-12,REQ-18" \
  --effort l

vcli --org "$ORG" work start AUTH-42
vcli --org "$ORG" work status AUTH-42 waiting
vcli --org "$ORG" work status AUTH-42 blocked
vcli --org "$ORG" work status AUTH-42 active
vcli --org "$ORG" work ready-for-review AUTH-42
vcli --org "$ORG" work complete AUTH-42
```

`work attach-session` requires a configured, running bridge. If it fails for
that reason, run `vcli service start`, confirm `vcli service status`, and retry
the attachment. It identifies the calling agent from its stable session ID or
process ancestry, reports it to the device, and attaches it idempotently to
Work. JSON output includes the real `liveActivityId`; retain that value for
`--execution`. Different local sessions can attach to the same Work and appear
as separate Work Sessions.

Valid effort values are `unknown`, `xs`, `s`, `m`, and `l`. Work may also take
`--project` and `--due-date` at creation.

## Watching Work

```bash
# Continuous human-readable watcher
vcli --org "$ORG" work watch AUTH-42

# Continuous NDJSON watcher with the full current snapshot first
vcli --org "$ORG" --json work watch AUTH-42 --events all --initial

# One bounded wakeup
vcli --org "$ORG" --json work watch AUTH-42 \
  --events work,tasks,requests,attention,handoffs,executions \
  --once --timeout 1800
```

Categories are `work`, `tasks`, `requests`, `attention`, `handoffs`, and
`executions`; `all` is the default. JSON mode emits one object per line.

## Handoffs and Attention

```bash
vcli --org "$ORG" work handoff AUTH-42 bob@example.com \
  --summary "Backend complete; admin UI remains" \
  --note "Resume at Task #4"

vcli --org "$ORG" work respond-handoff <handoffId> --accept true
vcli --org "$ORG" work respond-handoff <handoffId> --accept false

vcli --org "$ORG" work attention AUTH-42 \
  --title "Choose redirect behavior" \
  --details "Both options affect existing tenants" \
  --task 3 \
  --execution <liveActivityId>
```

The outgoing owner remains accountable until the incoming owner accepts.
Acceptance does not start the incoming ownership period; run `work start`.

## Tasks

```bash
vcli --org "$ORG" task list AUTH-42
vcli --org "$ORG" task create AUTH-42 --title "Build metadata endpoint"
vcli --org "$ORG" task create AUTH-42 \
  --title "Add integration tests" \
  --assignee alice@example.com \
  --execution <liveActivityId>

vcli --org "$ORG" task status AUTH-42 2 in_progress
vcli --org "$ORG" task status AUTH-42 2 waiting
vcli --org "$ORG" task status AUTH-42 2 blocked
vcli --org "$ORG" task status AUTH-42 2 done
vcli --org "$ORG" task assign AUTH-42 2 bob@example.com
vcli --org "$ORG" task assign AUTH-42 2
```

Use only an actual Vector live activity ID for `--execution`. Omit it when the
agent does not have one.

## GitHub Evidence

The current compatibility command uses the legacy Issue namespace because Work
and Issues share the underlying record:

```bash
vcli --org "$ORG" issue link-github AUTH-42 https://github.com/acme/app/pull/123
```

Run `vcli --org "$ORG" work context AUTH-42` afterward to verify the link.

## Legacy Issues

Use Request, Work, and Task for new workflows. Use Issue commands only for
compatibility or operations without a Work-named equivalent.

```bash
vcli --org "$ORG" issue list --project api --team eng --limit 50
vcli --org "$ORG" issue get API-1
vcli --org "$ORG" issue create --title "Ship CLI" --project api --team eng
vcli --org "$ORG" issue update API-1 --title "Ship CLI v2" --priority "Urgent"
vcli --org "$ORG" issue assign API-1 alice@example.com --state "In Progress"
vcli --org "$ORG" issue unassign API-1 alice@example.com
vcli --org "$ORG" issue comment API-1 --body "Ready for review"
vcli --org "$ORG" issue link-github API-1 https://github.com/acme/api/pull/123
vcli --org "$ORG" issue create --title "Sub-issue" --parent API-1 --project api
```
