# Agent Bridge

## Contents

- Purpose
- Service commands
- Execution behavior
- Local state
- Provider requirements

## Purpose

The bridge connects local developer machines to Vector. Codex, Claude Code,
Cursor, GitHub Copilot, OpenCode, Pi, and manual shell sessions can appear as
live executions on Work with bidirectional messaging.

Attaching or launching an execution does not start Work. Managed launch prompts
identify active Work and include linked Request context. The CLI syncs agent
events and session state back to Vector.

## Service Commands

```bash
vcli auth login
vcli service start
vcli service run
vcli service stop
vcli service status
vcli service install
vcli service uninstall
vcli service logs

vcli bridge start
vcli bridge stop
vcli bridge status
```

Use `vcli --profile work service start` to select the daemon account and make
that profile the default. `service start` also repairs the service after Node or
the global CLI installation moves.

## Execution Behavior

The service maintains a device heartbeat, polls commands, syncs agent events
and session state, discovers local processes, and updates the live activity
cache. Messages sent from a Work page can resume supported attached sessions.

When an agent starts acting on Work, attach the current session:

```bash
vcli --json work attach-session AUTH-42
vcli --json work attach-session AUTH-42 --task 3
```

The bridge must be running. If attachment says it is not configured or not
running, set it up and retry:

```bash
vcli service start
vcli service status
vcli --json work attach-session AUTH-42
```

Auto-detection prefers the stable agent session ID and then process ancestry;
use `--session <sessionKey>` only when the CLI reports ambiguity. Repeating the
command for the same active session is safe. Each distinct agent session creates
its own Work Session, and supported Codex and Claude sessions can receive
messages from the web and iOS Work pages.

Pass the returned `liveActivityId` to Task and attention commands when available.
This records agent provenance and applies the Work agent policy.

The macOS menu bar lists bridge status and active agent sessions with Work keys.
Build it with `pnpm --filter @rehpic/vcli build`; open the generated app under
`packages/vector-cli/native/VectorMenuBar.app`.

## Local State

Bridge state is stored under `~/.vector/`:

| File                   | Purpose                                 |
| ---------------------- | --------------------------------------- |
| `bridge.json`          | Device identity, secret, and Convex URL |
| `bridge.pid`           | Running service PID                     |
| `bridge.log`           | Service output in LaunchAgent mode      |
| `live-activities.json` | Cached sessions for the menu bar        |
| `cli-default.json`     | Default-profile authentication session  |

## Provider Requirements

- Codex requires the `codex` CLI.
- Claude requires credentials usable by the Claude Agent SDK.
- Cursor uses `cursor-agent` for the CLI-owned fallback.
- Copilot uses local Copilot CLI/SDK credentials.
- OpenCode requires the `opencode` CLI.
- Pi requires the local `pi` CLI.
- Manual shell sessions use the internal `vector_cli` provider.
