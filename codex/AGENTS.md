# Global Safety Rules

These rules apply to all Codex sessions unless the user explicitly overrides them in the current conversation.

## Confirm Before Destructive Changes

- Before deleting files, directories, or large blocks of content, list the exact targets and wait for explicit user confirmation.
- This includes recursive deletes, wildcard deletes, generated-output cleanup, and removals outside the current project.
- Do not treat "clean up" or "remove unused files" as permission to delete without first showing the deletion list.

## Confirm Before Irreversible Or High-Risk Commands

- Ask for confirmation before running `git push --force`, `git push --force-with-lease`, `git reset --hard`, history rewrites, recursive deletes, disk formatting, package uninstall commands, or commands that overwrite global/system configuration.
- Ask for confirmation before modifying authentication files, shell profiles, SSH keys, Git credentials, package manager global config, or files under user-wide tool configuration directories unless the user specifically requested that exact edit.
- Prefer non-destructive alternatives first, such as listing, dry runs, backups, or status checks.

## Network Access Policy

- Before running network commands, explain the purpose and wait for user confirmation.
- Network commands include package installs or updates, `curl`, `wget`, `Invoke-WebRequest`, `iwr`, `git fetch`, `git pull`, `git push`, `npm install`, `pnpm install`, `pip install`, `uv sync`, `cargo install`, and similar commands.
- If a command unexpectedly needs network access, stop and ask before retrying with network-dependent behavior.

## Bypass And Sandbox Awareness

- Even when filesystem sandboxing is disabled or approvals are reduced, keep these confirmation rules in force.
- Treat hooks and these instructions as safety guardrails, not as permission to bypass user intent.

## Key File Output Reporting

- Whenever a task involves key file outputs, the final response must list each key output file in a list format.
- For each key output file, include:
  - Output path
  - File purpose
