# Vector CLI Task Management Skill

AI agent skill for the [Vector CLI](https://github.com/xrehpicx/vector) (`vcli`) — a command-line interface for [Vector](https://github.com/xrehpicx/vector), an open-source project management platform.

When installed, this skill is identified as `task-management`.

## Install the Skill

```bash
npx skills add xrehpicx/vector-skill
```

## Install the CLI

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

Requires Node.js >= 20.19.0.

## What This Skill Covers

This skill teaches AI agents how to use `vcli`. It covers:

- **Authentication** — signup, login, logout, session management, profiles
- **Organizations** — create, manage members, invites, roles
- **Teams & Projects** — CRUD, member management, leads
- **Issues** — create, assign, comment, priorities, states, time estimates, sub-issues
- **Documents & Folders** — create, update, move, organize
- **Roles (RBAC)** — custom roles, permissions, assignment
- **Notifications** — inbox, preferences, push subscriptions
- **Workspace Discovery** — reference data, global search, permission checks
- **Activity Feeds** — per project, team, issue, or document
- **Issue States, Priorities, Project Statuses** — customize metadata
- **Platform Admin** — branding, signup policies
- **Scripting** — JSON output, profiles, automation patterns
- **Troubleshooting** — common errors and fixes

## Quick Start

```bash
vcli --app-url http://localhost:3000 auth signup \
  --email you@example.com --username you --password 'secret'

vcli org create --name "Acme" --slug acme
vcli org use acme

vcli team create --key eng --name "Engineering"
vcli project create --key api --name "API" --team eng
vcli issue create --title "First issue" --project api --team eng
```

## Links

- [Vector repo](https://github.com/xrehpicx/vector)
- [CLI package on npm](https://www.npmjs.com/package/@rehpic/vcli)
- [CLI README](https://github.com/xrehpicx/vector/blob/main/packages/vector-cli/README.md)
