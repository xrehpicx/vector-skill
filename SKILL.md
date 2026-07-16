---
name: task-management
description: Request, Work, and Task management skill for Vector CLI (vcli), including agent context, execution attribution, handoffs, review, notifications, and legacy Issue compatibility.
---

# Vector CLI

Use this skill when a user asks how to use the Vector CLI (`vcli`), needs command examples, or wants to manage their Vector workspace from the terminal.

Vector is an open-source coordination platform for human and agent work. The CLI gives you full access to Requests, Work, Tasks, organizations, teams, projects, documents, roles, notifications, and more.

The primary delivery model is:

- **Request**: intake, required expected output, routing, and requester review.
- **Work**: durable outcome context with a live workpad, ownership periods, linked Requests, Tasks, agent executions, and GitHub evidence.
- **Task**: an optional tracked step within Work. Do not create Tasks for every checklist line; the Work workpad can remain unstructured.

Assignment, Request acceptance, and agent attachment never start Work. Starting is an explicit human action through `vcli work start <key>` or `vcli work status <key> active`.

## Installation

```bash
# npm
npm install -g @rehpic/vcli

# pnpm
pnpm add -g @rehpic/vcli

# yarn
yarn global add @rehpic/vcli

# bun
bun add -g @rehpic/vcli
```

Requires Node.js >= 22.19.0.

After installing, the `vcli` binary is available globally.

## Global Options

Every command accepts these flags:

| Flag                 | Description                                                                         |
| -------------------- | ----------------------------------------------------------------------------------- |
| `--app-url <url>`    | Vector app URL (or set `NEXT_PUBLIC_APP_URL`)                                       |
| `--convex-url <url>` | Convex deployment URL (auto-resolved from app URL, or set `NEXT_PUBLIC_CONVEX_URL`) |
| `--org <slug>`       | Organization slug override                                                          |
| `--profile <name>`   | CLI profile (default: `default`)                                                    |
| `--json`             | Output as JSON for scripting                                                        |

## List Filtering, Sorting, and Links

Legacy and supporting entity list commands (`issue list`, `project list`, `team list`, `document list`, `folder list`) support:

| Flag                      | Description                                              |
| ------------------------- | -------------------------------------------------------- |
| `--limit <n>`             | Maximum number of results                                |
| `--created-after <date>`  | Filter: created on or after date (ISO format)            |
| `--created-before <date>` | Filter: created on or before date (ISO format)           |
| `--sort <field>`          | Sort by field (e.g. `createdAt`, `title`, `name`, `key`) |
| `--order <direction>`     | Sort direction: `asc` or `desc` (default: `asc`)         |

Documents also support `--updated-after` and `--updated-before` for filtering by last edited time.

Request and Work lists use purpose-built scopes plus `--limit`:

| Command             | Valid scopes                            | Default  |
| ------------------- | --------------------------------------- | -------- |
| `vcli request list` | `inbox`, `mine`, `requested`, or `all`  | `inbox`  |
| `vcli work list`    | `active`, `mine`, `attention`, or `all` | `active` |

They do not currently expose the generic date and sort flags in the table above.

Request and Work list output, along with the generic entity lists above,
include a `url` field linking to each entity in the web app.

```bash
# Active Work across the workspace
vcli work list --scope active

# Work you own
vcli work list --scope mine

# Top 5 projects by name
vcli project list --sort name --limit 5

# Documents edited since a date
vcli document list --updated-after 2025-03-01 --sort lastEditedAt --order desc

# Teams created before a date
vcli team list --created-before 2025-01-01 --limit 10
```

## Core Concepts

### Convex URL Auto-Resolution

You typically only need to provide `--app-url`. The CLI automatically fetches the Convex deployment URL from the app's `/api/config` endpoint when it's not specified via `--convex-url`, a saved session, or environment variables. The resolved URL is cached in your profile for subsequent commands.

### Profiles

Profiles let you keep separate CLI sessions on one machine. Each profile stores its session at `~/.vector/cli-<profile>.json`.

```bash
# Use a named profile
vcli --profile work auth login you@example.com --password 'secret'
vcli --profile staging --app-url https://staging.example.com auth whoami
```

### Organization Context

Most commands require an active organization. You can either:

- Pass `--org <slug>` on every command, or
- Set it once with `vcli org use <slug>`

### Member Resolution

Anywhere a `<member>` argument is needed, you can use an email address, username, or user ID.

### Date Format

Dates use ISO format: `YYYY-MM-DD` or full ISO 8601 datetime.

---

## Authentication

### Device Flow (default)

The default login opens your browser for approval — no password needed in the terminal:

```bash
# Opens browser, shows code, waits for approval
vcli auth login
```

The CLI will:

1. Request a device code from the server
2. Open your browser to the approval page with the code pre-filled
3. Show the URL and code as a fallback if the browser doesn't open
4. Poll until you approve in the browser

### Password Login

Pass an identifier and/or `--password` to use direct password auth:

```bash
vcli auth login you@example.com --password 'secret'
vcli auth login yourname --password 'secret'
```

### Other Auth Commands

```bash
# Sign up
vcli auth signup --email you@example.com --username yourname --password 'secret'

# Check current session
vcli auth whoami

# Log out
vcli auth logout
```

`whoami` shows the current user, org memberships, and active org.

---

## Organizations

```bash
# List your organizations
vcli org list

# Create a new org
vcli org create --name "Acme" --slug acme

# Set active org
vcli org use acme

# Show active org
vcli org current

# Update org details
vcli org update acme --name "Acme Corp" --new-slug acme-corp

# Get org statistics
vcli org stats acme

# Upload org logo
vcli org logo acme --file ./logo.png

# List members (includes roles)
vcli org members acme

# Invite a member
vcli org invite acme --email teammate@example.com --role admin

# Change a member's role
vcli org member-role teammate@example.com --role admin

# Remove a member
vcli org remove-member teammate@example.com

# List pending invites
vcli org invites acme

# Revoke an invite
vcli org revoke-invite <inviteId>
```

---

## Invitations

Manage invitations from the invited user's perspective:

```bash
# List your pending invitations
vcli invite list

# Accept an invite
vcli invite accept <inviteId>

# Decline an invite
vcli invite decline <inviteId>
```

---

## Teams

```bash
# List teams (supports --limit, --created-after/before, --sort, --order)
vcli team list

# Get team details
vcli team get eng

# Create a team
vcli team create --key eng --name "Engineering" --description "Core engineering"

# Update a team
vcli team update eng --name "Platform Engineering" --icon code --color "#3B82F6"

# Delete a team
vcli team delete eng

# Manage members
vcli team members eng
vcli team add-member eng alice@example.com --role lead
vcli team remove-member eng alice@example.com
vcli team set-lead eng alice@example.com
```

---

## Projects

```bash
# List projects (supports --limit, --created-after/before, --sort, --order)
vcli project list --team eng

# Get project details
vcli project get api

# Create a project
vcli project create --key api --name "API" --team eng --status "In Progress"

# Update a project
vcli project update api --description "Backend API" --due-date 2025-06-01

# Delete a project
vcli project delete api

# Manage members
vcli project members api
vcli project add-member api bob@example.com --role lead
vcli project remove-member api bob@example.com
vcli project set-lead api bob@example.com
```

Clear optional fields with `--clear-*` flags (e.g., `--clear-team`, `--clear-due-date`).

---

## Requests

Every Request requires an expected output. Route it to one or more people, claim it for planning, and attach one or more Work records. None of these actions starts Work.

```bash
# Inspect intake
vcli request list --scope inbox
vcli request list --scope requested
vcli request get REQ-12

# Create and route an outcome-oriented Request
vcli request create \
  --title "Add enterprise SSO" \
  --description "Customer needs SAML for the admin console" \
  --expected-output "Admins can configure SAML and users can sign in through it" \
  --review-guidance "Test with the customer's metadata file" \
  --recipients "alice@example.com,bob@example.com"

vcli request route REQ-12 "alice@example.com,bob@example.com"
vcli request claim REQ-12

# A Request can be fulfilled by existing or newly created Work.
# `fulfills` participates in the requester review roll-up; `contributes`
# attaches useful context without blocking that roll-up.
vcli request link-work REQ-12 AUTH-42 --relation fulfills
vcli request link-work REQ-12 DOCS-7 --relation contributes

# Requester review loop
vcli request request-changes REQ-12 --note "SCIM provisioning is still missing"
vcli request complete REQ-12 --note "Validated with the customer configuration"
```

## Work

Work is the primary execution context. It can deliver several related Requests and hold many Tasks or only an unstructured workpad. Multiple Work records may be active for the same person.

```bash
vcli work list --scope mine
vcli work list --scope attention
vcli work get AUTH-42

vcli work create \
  --title "Enterprise identity rollout" \
  --workpad "Notes, decisions, and a live checklist" \
  --owner alice@example.com \
  --request "REQ-12,REQ-18" \
  --effort l

# Explicit intent: assignment alone is not execution
vcli work start AUTH-42

# Pause/resume and make blockers visible
vcli work status AUTH-42 waiting
vcli work status AUTH-42 blocked
vcli work status AUTH-42 active

# Fetch bounded context before an agent acts
vcli work context AUTH-42
vcli work context AUTH-42 --task 3

# Subscribe to only the changes this agent is waiting for
vcli work watch AUTH-42 --events tasks,requests

# Ask a human to look. Agents should pass their live execution id.
vcli work attention AUTH-42 \
  --title "Choose redirect behavior" \
  --details "Both options affect existing tenants" \
  --task 3 \
  --execution <liveActivityId>

# The current owner stays accountable until the next owner accepts.
vcli work handoff AUTH-42 bob@example.com \
  --summary "SAML backend is complete; admin UI remains" \
  --note "Start from Task #4"

# The incoming owner runs this. Get the pending handoff id from their
# notification, from the proposer, or from `vcli work context AUTH-42`
# (the `handoffs` entries include their ids).
vcli work respond-handoff <handoffId> --accept true

# The incoming owner also runs this: acceptance does not start their ownership
# period. Start it explicitly even when aggregate Work stayed active.
vcli work start AUTH-42

vcli work ready-for-review AUTH-42
vcli work complete AUTH-42
```

`work context` returns linked Requests and their expected outputs, Tasks, ownership and handoff history, blockers, open attention requests, GitHub development evidence, recent activity, and attached executions. Prefer it over assembling context from unrelated workspace searches.

### Watch Work for changes

Use a real-time watcher when an agent should wait for another actor, Task, or
Request instead of repeatedly polling `work context`:

```bash
# Human-readable continuous stream; Ctrl+C stops it
vcli work watch AUTH-42

# Wait for one relevant change, emit one NDJSON object, and bound the wait
vcli --json work watch AUTH-42 \
  --events tasks,requests,attention \
  --once \
  --timeout 1800
```

Categories are `work`, `tasks`, `requests`, `attention`, `handoffs`, and
`executions`; `all` is the default. Events are granular, including
`task.completed`, `request.added`, and the corresponding `added`, `updated`,
and `removed` events. Global `--json` uses NDJSON: each new event is a single
JSON line, not one long-lived JSON array. Add `--initial` only when the agent
also needs the complete current snapshot before waiting.

Prefer `work watch` over a shell polling loop. Use `--once` for a single wakeup
and omit it when the agent needs a continuous event stream.

## Tasks

Tasks are optional tracked units under Work. Use them when assignment, status, a blocker, or agent attribution matters; keep lightweight notes and checklists in the Work workpad.

```bash
vcli task list AUTH-42
vcli task create AUTH-42 --title "Build metadata endpoint" --assignee alice@example.com

# Agents must pass their actual Vector live activity id. This records agent
# attribution and enforces the Work's allow / approval_required / deny policy.
vcli task create AUTH-42 \
  --title "Add SAML integration tests" \
  --execution <liveActivityId>

vcli task status AUTH-42 2 in_progress
vcli task status AUTH-42 2 waiting
vcli task status AUTH-42 2 blocked
vcli task status AUTH-42 2 done
vcli task assign AUTH-42 2 bob@example.com
vcli task assign AUTH-42 2  # Clear assignee
```

Never invent an execution id or attribute a human-created Task to an agent. If no live execution is available, omit `--execution`; the Task is recorded as human/CLI-created.

The CLI and backend expose this identifier as a live activity id
(`liveActivityId`); product UI and this skill call the same record an agent
execution.

---

## Legacy Issues

Issue commands remain for backwards compatibility while existing integrations migrate. For new workflows, use Request, Work, and Task commands above.

### CRUD

```bash
# List issues (supports --limit, --created-after/before, --sort, --order)
vcli issue list --project api --team eng --limit 50

# Get issue details
vcli issue get API-1

# Create an issue
vcli issue create \
  --title "Ship CLI" \
  --project api \
  --team eng \
  --priority "High" \
  --assignee alice@example.com \
  --state "In Progress" \
  --due-date 2025-04-01

# Update an issue
vcli issue update API-1 --title "Ship CLI v2" --priority "Urgent"

# Delete an issue
vcli issue delete API-1
```

### Assignments

```bash
# Assign a member
vcli issue assign API-1 alice@example.com --state "In Progress"

# Unassign a member
vcli issue unassign API-1 alice@example.com

# List all assignments
vcli issue assignments API-1

# Replace all assignees at once
vcli issue replace-assignees API-1 alice@example.com,bob@example.com

# Manage individual assignments
vcli issue set-assignment-state <assignmentId> <state>
vcli issue reassign-assignment <assignmentId> bob@example.com
vcli issue remove-assignment <assignmentId>
```

### Other Issue Operations

```bash
# Set priority directly
vcli issue set-priority API-1 "High"

# Set time estimates per state
vcli issue set-estimates API-1 --values "todo=4,in_progress=8"

# Add a comment
vcli issue comment API-1 --body "Looks good, shipping tomorrow"

# Link a GitHub PR, issue, or commit by URL
vcli issue link-github API-1 "https://github.com/acme/api/pull/123"

# Link the same PR to another issue when one PR covers multiple issues
vcli issue link-github API-2 "https://github.com/acme/api/pull/123"

# Create a sub-issue
vcli issue create --title "Sub-task" --parent API-1 --project api
```

## Agent Bridge Service

The bridge connects local developer machines to Vector. Local Codex, Claude Code, Cursor, GitHub Copilot, OpenCode, and Pi sessions show up as **live executions** on Work with bidirectional messaging. Attaching or launching an execution does not change the Work status. Managed launch prompts identify the active Vector Work, include its linked Request context, and keep the agent inside that Work scope. For managed launches, the CLI owns the provider session directly and syncs agent events back to Convex; terminal/tmux integration is still available for attached shell sessions.

### Starting the bridge

```bash
# Log in first
vcli auth login

# Start the bridge service (registers device, repairs the LaunchAgent, and restarts it)
vcli service start

# Select the daemon account explicitly; this also becomes the default profile
vcli --profile work service start

# Or install as a macOS LaunchAgent (auto-starts on login)
vcli bridge start
```

### Service management

```bash
vcli service start      # Register the device and start the background service
vcli service run        # Run the bridge in the foreground
vcli service stop       # Stop the bridge
vcli service status     # Show running, degraded, starting, or stopped status
vcli service install    # Install as macOS LaunchAgent
vcli service uninstall  # Remove LaunchAgent
vcli service logs       # Tail bridge logs

vcli bridge start       # Shortcut: register + install + start
vcli bridge stop        # Stop + uninstall
vcli bridge status      # Quick status check
```

### How it works

The bridge runs as a local Node.js process using `ConvexHttpClient`:

- **Heartbeat** (30s): Keeps the device marked as online
- **Command polling** (5s): Picks up messages sent from the Vector Work page
- **Agent event sync**: Sends assistant, reasoning, tool, status, auth, compaction, and error events from local agent sessions back to Convex
- **Cells-style session state**: Syncs model, permission mode, thinking level, fast mode, context length, queued messages, pending approvals, pending questions, plans, and usage through Convex
- **Process discovery** (60s): Finds local agent/tmux processes via `ps`
- **Live activity cache** (5s): Writes `~/.vector/live-activities.json` for the macOS menu bar

### macOS menu bar

A native Swift status bar app shows the Vector icon with:

- Bridge status (running/offline)
- List of active agent sessions with Work keys (click to open in Vector)
- Start/Stop/Restart controls

```bash
# Build (requires Xcode CLI tools)
pnpm --filter @rehpic/vcli build

# Run
open packages/vector-cli/native/VectorMenuBar.app
```

Auto-installed as a LaunchAgent via `vcli service install`.

The menu bar uses the active CLI profile. Selecting another profile restarts
and reconciles the bridge. Automatic updates are opt-in and follow the stable
`latest` npm channel; use `vcli update` to update explicitly. Running
`vcli service start` is also the repair command after Node or the global CLI
installation moves.

### Configuration

All bridge state lives in `~/.vector/`:

| File                   | Purpose                          |
| ---------------------- | -------------------------------- |
| `bridge.json`          | Device ID, secret, convex URL    |
| `bridge.pid`           | Running bridge PID               |
| `bridge.log`           | Bridge stdout (LaunchAgent mode) |
| `live-activities.json` | Cached sessions for menu bar     |
| `cli-default.json`     | CLI auth session                 |

### Architecture reference

See `docs/architecture/agent-device-bridge/README.md` for the full data model, flows, and security details.

### Agent branding

- Internal provider key: `codex` — visible label: `Codex`
- Internal provider key: `claude_code` — visible label: `Claude` or `Claude Agent`
- Internal provider key: `cursor` — visible label: `Cursor`
- Internal provider key: `copilot` — visible label: `GitHub Copilot`
- Internal provider key: `opencode` — visible label: `OpenCode`
- Internal provider key: `pi` — visible label: `Pi`
- Internal provider key: `vector_cli` — visible label: `Vector CLI` for manual shell sessions
- Final completion comments include `authorKind: 'agent'` and `agentSource` metadata

Provider requirements:

- Codex requires the `codex` CLI.
- Claude requires Claude credentials usable by the Claude Agent SDK.
- Cursor uses `cursor-agent` for the CLI-owned fallback path.
- Copilot uses the local Copilot CLI/SDK credentials.
- OpenCode uses the `opencode` CLI.
- Pi uses the local `pi` CLI when available.

---

## Documents

```bash
# List documents (supports --limit, --created-after/before, --updated-after/before, --sort, --order)
vcli document list --folder-id <folderId>

# Get document details
vcli document get <documentId>

# Create a document
vcli document create \
  --title "Architecture Overview" \
  --content "# Overview\n\nThis document covers..." \
  --team eng \
  --project api

# Update a document
vcli document update <documentId> --title "Updated Title" --content "New content"

# Move to a folder
vcli document move <documentId> --folder-id <folderId>

# Delete a document
vcli document delete <documentId>
```

---

## Folders

```bash
# List folders
vcli folder list

# Create a folder
vcli folder create --name "Engineering Docs" --description "Internal docs"

# Update a folder
vcli folder update <folderId> --name "Eng Docs"

# Delete a folder
vcli folder delete <folderId>
```

---

## Roles (RBAC)

```bash
# List organization roles
vcli role list

# Get role details and permissions
vcli role get "Engineering Lead"

# Create a custom role
vcli role create \
  --name "Triage Lead" \
  --permissions "issue:create,issue:update,issue:assign" \
  --description "Can triage and assign issues"

# Update a role
vcli role update "Triage Lead" \
  --name "Triage Manager" \
  --permissions "issue:create,issue:update,issue:assign,issue:delete"

# Assign/unassign roles to members
vcli role assign "Triage Lead" alice@example.com
vcli role unassign "Triage Lead" alice@example.com
```

---

## Notifications

```bash
# View inbox
vcli notification inbox --filter unread --limit 10

# Get unread count
vcli notification unread-count

# Mark as read
vcli notification mark-read <recipientId>
vcli notification mark-all-read

# Archive a notification
vcli notification archive <recipientId>

# View/update notification preferences
vcli notification preferences
vcli notification set-preference assignments --in-app true --email true --push false

# Manage push subscriptions
vcli notification subscriptions
vcli notification remove-subscription <subscriptionId>
```

---

## Workspace Discovery

### Reference Data

Get all workspace metadata (states, priorities, statuses, members, teams, projects) in one call:

```bash
vcli refdata acme
```

### Global Search

```bash
vcli search "billing" --limit 10
```

Searches across issues, teams, projects, and documents.

### Permission Checks

```bash
# Check a single permission
vcli permission check issue:create --team eng --project api

# Check multiple permissions at once
vcli permission check-many "issue:create,issue:update,project:delete" --team eng
```

### Icon Search

```bash
vcli icons "check" --limit 5
```

Searches Lucide icons by name.

---

## Activity Feed

```bash
vcli activity project api --limit 10
vcli activity team eng --limit 10
vcli activity issue API-1 --limit 10
vcli activity document <documentId> --limit 10
```

All activity commands support `--cursor` for pagination.

---

## User Presence & Custom Status

Discord-like presence and custom status for your user profile.

### Presence

```bash
# View your current status
vcli presence get

# Set presence (online, idle, dnd, invisible)
vcli presence set online
vcli presence set dnd
vcli presence set invisible
```

### Custom Status

```bash
# Set a custom status with emoji and text
vcli presence custom --emoji "🚀" --text "Shipping features"

# Set with auto-clear (30m, 1h, 4h, today, never)
vcli presence custom --emoji "☕" --text "On a break" --clear-after 30m
vcli presence custom --emoji "🏠" --text "Working from home" --clear-after today

# Clear custom status
vcli presence clear
```

Presence determines the colored dot on your avatar: green (online), amber (idle), red (dnd), gray (invisible/offline). Custom status shows your emoji and text to other org members.

---

## Issue States, Priorities, and Project Statuses

Customize your workspace metadata:

```bash
# States (for issues)
vcli state list
vcli state create --name "In Review" --position 3 --type in_progress --color "#F59E0B" --icon eye
vcli state update <state> --name "Reviewing" --position 3 --type in_progress --color "#F59E0B"
vcli state update <state> --name "Reviewing" --position 3 --type in_progress --color "#F59E0B" --clear-icon
vcli state delete <state>
vcli state reset  # Reset to defaults

# Priorities (for issues)
vcli priority list
vcli priority create --name "Critical" --weight 5 --color "#EF4444" --icon alert-triangle
vcli priority update <priority> --name "Blocker" --color "#DC2626"
vcli priority update <priority> --name "Blocker" --color "#DC2626" --clear-icon
vcli priority delete <priority>
vcli priority reset  # Reset to defaults

# Statuses (for projects)
vcli status list
vcli status create --name "On Hold" --position 3 --type planned --color "#6B7280" --icon pause
vcli status update <status> --name "Paused" --position 3 --type planned --color "#6B7280"
vcli status update <status> --name "Paused" --position 3 --type planned --color "#6B7280" --clear-icon
vcli status delete <status>
vcli status reset  # Reset to defaults
```

---

## Platform Admin

Requires platform-admin privileges:

```bash
# Branding
vcli admin branding
vcli admin set-branding --name "My Platform" --theme-color "#1E40AF" --logo ./logo.png

# Signup policy
vcli admin signup-policy
vcli admin set-signup-policy --blocked "spam.com,trash.com" --allowed "company.com"

# Sync disposable email domains
vcli admin sync-disposable-domains
```

---

## Common Workflows

### First-Run Setup

```bash
vcli auth signup --email you@example.com --username you --password 'secret'
vcli org create --name "Acme" --slug acme
vcli org use acme
vcli team create --key eng --name "Engineering"
vcli project create --key api --name "API" --team eng
REQUEST_KEY="$(vcli --json request create \
  --title "First request" \
  --expected-output "A delivered, reviewable outcome" | jq -r '.requestKey')"
vcli work create --title "Deliver the first request" --request "$REQUEST_KEY"
vcli auth whoami
```

### Invite and Onboard a Teammate

```bash
# You (org admin)
vcli org invite acme --email teammate@example.com --role member

# Teammate
vcli auth signup --email teammate@example.com --username teammate --password 'secret'
vcli invite list
vcli invite accept <inviteId>
vcli org use acme
```

### Discover Workspace Before Scripting

```bash
vcli refdata acme                          # See all states, priorities, teams, projects
vcli search --org acme "billing"           # Find entities
vcli permission check issue:create         # Verify permissions
```

### Script With JSON Output

```bash
# Pipe to jq for structured processing
vcli --json work list --org acme | jq '.[].title'
vcli --json notification inbox --filter unread | jq '.page | length'
```

Tips for scripting:

- Always use `--json` for parseable output
- Always use explicit `--profile` and `--org` flags
- Use `refdata` to discover valid keys before mutations

---

## Troubleshooting

| Error                              | Fix                                                                                                         |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `Not logged in`                    | Run `vcli auth login` or `vcli auth signup`                                                                 |
| `app URL is required`              | Pass `--app-url <url>`, set `NEXT_PUBLIC_APP_URL`, or log in once with `--app-url` so the profile stores it |
| `Organization slug is required`    | Pass `--org <slug>` or run `vcli org use <slug>`                                                            |
| Auth errors against wrong server   | Verify `--app-url` matches the running app origin                                                           |
| Convex connection errors           | Usually auto-resolved from the app URL. If not, check `NEXT_PUBLIC_CONVEX_URL` or pass `--convex-url`       |
| Validation errors on create/update | Check keys, slugs, and required options. Use `vcli refdata` to discover valid values first                  |
