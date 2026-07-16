# Vector CLI Operations

## Contents

- Installation and context
- Authentication
- Organizations and invitations
- Teams and projects
- Documents and folders
- Roles and notifications
- Discovery and metadata
- Presence and administration
- Scripting and troubleshooting

## Installation and Context

Install `@rehpic/vcli` globally with npm, pnpm, yarn, or bun. Node.js 22.19 or
newer is required.

Global flags are `--app-url`, `--convex-url`, `--org`, `--profile`, and `--json`.
The CLI normally resolves the Convex deployment from the app URL and caches it
in the profile. Use `vcli org use <slug>` to set the active organization.

## Authentication

```bash
vcli auth login
vcli auth login you@example.com --password 'secret'
vcli auth signup --email you@example.com --username you --password 'secret'
vcli auth whoami
vcli auth logout
```

## Organizations and Invitations

```bash
vcli org list
vcli org create --name "Acme" --slug acme
vcli org use acme
vcli org current
vcli org update acme --name "Acme Corp"
vcli org stats acme
vcli org members acme
vcli org invite acme --email teammate@example.com --role member
vcli org member-role teammate@example.com --role admin
vcli org remove-member teammate@example.com
vcli org invites acme
vcli org revoke-invite <inviteId>

vcli invite list
vcli invite accept <inviteId>
vcli invite decline <inviteId>
```

## Teams and Projects

```bash
vcli team list
vcli team get eng
vcli team create --key eng --name "Engineering"
vcli team update eng --name "Platform Engineering"
vcli team members eng
vcli team add-member eng alice@example.com --role lead
vcli team remove-member eng alice@example.com
vcli team set-lead eng alice@example.com

vcli project list --team eng
vcli project get api
vcli project create --key api --name "API" --team eng
vcli project update api --description "Backend API"
vcli project members api
vcli project add-member api bob@example.com --role lead
vcli project remove-member api bob@example.com
vcli project set-lead api bob@example.com
```

## Documents and Folders

```bash
vcli document list --folder-id <folderId>
vcli document get <documentId>
vcli document create --title "Architecture" --content "# Overview" --team eng
vcli document update <documentId> --title "Updated title"
vcli document move <documentId> --folder-id <folderId>
vcli document delete <documentId>

vcli folder list
vcli folder create --name "Engineering Docs"
vcli folder update <folderId> --name "Eng Docs"
vcli folder delete <folderId>
```

## Roles and Notifications

```bash
vcli role list
vcli role get "Engineering Lead"
vcli role create --name "Triage Lead" --permissions "issue:create,issue:update"
vcli role update "Triage Lead" --permissions "issue:create,issue:update,issue:assign"
vcli role assign "Triage Lead" alice@example.com
vcli role unassign "Triage Lead" alice@example.com

vcli notification inbox --filter unread --limit 10
vcli notification unread-count
vcli notification mark-read <recipientId>
vcli notification mark-all-read
vcli notification archive <recipientId>
vcli notification preferences
vcli notification subscriptions
```

## Discovery and Metadata

```bash
vcli refdata acme
vcli search "billing" --limit 10
vcli permission check issue:create --team eng --project api
vcli permission check-many "issue:create,issue:update,project:delete"
vcli icons "check" --limit 5

vcli activity project api --limit 10
vcli activity team eng --limit 10
vcli activity issue API-1 --limit 10
vcli activity document <documentId> --limit 10
```

Use `refdata` before guessing keys, states, priorities, statuses, teams, or
projects. Activity commands accept cursors for pagination.

## Presence and Administration

```bash
vcli presence get
vcli presence set online
vcli presence set dnd
vcli presence custom --emoji "🚀" --text "Shipping" --clear-after 1h
vcli presence clear

vcli state list
vcli priority list
vcli status list

vcli admin branding
vcli admin signup-policy
```

Administration commands require platform-admin privileges.

## Scripting and Troubleshooting

Use `--json` for parseable output and explicit `--profile` plus `--org` in
scripts. Request and Work lists have purpose-built `--scope` and `--limit`
options. Supporting entity lists accept date filters, sorting, and ordering.

Common fixes:

| Error                   | Action                                        |
| ----------------------- | --------------------------------------------- |
| Not logged in           | Run `vcli auth login`                         |
| App URL required        | Pass `--app-url` or set `NEXT_PUBLIC_APP_URL` |
| Organization required   | Pass `--org` or run `vcli org use`            |
| Wrong-server auth error | Verify the app URL and selected profile       |
| Convex connection error | Verify auto-resolution or pass `--convex-url` |
| Validation error        | Use `vcli refdata` and check required options |
