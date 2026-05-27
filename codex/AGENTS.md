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

## Global Configuration Sync

- When Codex actually modifies global configuration or user-wide rule files, it must check whether related config-prompt files are still accurate.
- Related config-prompt files include `codex/AGENTS.md`, `codex/my-global-config-prompt.md`, and `codex/config.toml.example` when the change affects their documented behavior.
- Do not perform this check mechanically when no global configuration or user-wide rule file was modified.
- If config-prompt files need updates, update them locally and create a git commit.
- Before pushing to a remote repository, explain the network action and wait for user confirmation unless the user has already explicitly authorized that push in the current context.

## Key File Output Reporting

- Whenever a task involves key file outputs, the final response must list each key output file in a list format.
- For each key output file, include:
  - Output path
  - File purpose

## Skill Organization Policy

- Keep root-level active skills focused and intentional to avoid skills context budget pressure.
- When a product, platform, method, or workflow family has two or more related skills, prefer one root-level grouped skill with child skills under `library/`.
- Add a new skill to an existing group when it clearly belongs to that domain, and update the group skill's `SKILL.md` child workflow list.
- Create a new grouped skill when the capability is likely to grow into a family.
- Put a skill directly in the active root only when it is independent, broadly useful, or a foundational capability with no clear parent group.
- Do not use `skills.disabled` as the primary grouping mechanism. Treat it only as an archive or temporary disabled area when explicitly needed.
- Examples: `gsd`, `superpowers`, `vercel`, `supabase`, `atlassian`, `mcp-server-dev`, `telegram`, and `slack` should be grouped skills with internal `library/` child workflows.

## DOCX Safety Policy

- When editing `.docx` academic or thesis documents, use the `docx` skill and follow its conservative ZIP/XML editing and validation workflow.
- Do not flatten complex Word runs, formulas, field codes, bookmarks, drawings, or cross-references.
- Before declaring DOCX edits complete, run structural validation and report key validation results.
