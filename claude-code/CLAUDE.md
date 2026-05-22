# Personal preferences (global)

These preferences apply to **all** Claude Code sessions on this machine, regardless of project.

## Always report output paths

After any Write/Edit/script run that changes a file the user cares about, **list the absolute file path** in the reply.

- Use absolute Windows paths (e.g. `D:\...`, `C:\...`), not relative.
- For multi-file output (backup + main + previews), use a small table: file / path / operation.
- Skip trivial scratch files (one-off inspection scripts the user won't open).

**Why:** User asked explicitly after past sessions where modified files were left unannounced and she had to ask "this is your final output?" to find them.

## Confirm before delete

Before deleting any file or directory, **first enumerate the targets** (absolute paths) and wait for explicit user confirmation. Never auto-delete.

- Applies to `rm`, `Remove-Item`, `del`, batch cleanup scripts — no matter how "safe" it looks.
- For multi-file deletes, use a table: path / type / reason / confirmed.
- Even routine temp dirs like `__pycache__` get listed first.

**Why:** Project roots tend to mix temp scripts, backups, intermediate artifacts and real outputs. Mis-delete risk is high enough that the rule is absolute.

## Network operations require confirmation

Any operation that sends data over the network — `git push`, posting messages (Slack/email/IM), calling external APIs, uploading files, opening PRs/issues — **requires explicit per-time confirmation**, even when the surrounding workflow is otherwise auto-approved. Local operations (file edits, `git add`, `git commit`, running tests/scripts) do not.

**Why:** Network ops are visible to others, often irreversible, and frequently authoritative (a push lands in shared history; a message can't be unsent). Local ops can be safely undone before they leave the machine.

**How to apply:**
- Before any network call, surface what's about to happen (target URL/repo/recipient + a one-line payload summary) and wait for explicit go-ahead.
- A standing authorization for a specific repo/workflow only authorizes the *destination*, never bypasses this confirmation step.
- Apply this every time, not just on the first push of a session. The user should never be surprised by an outbound request.

## Sync global-config files on user-global changes

Whenever the user makes a change in the **"user-global" / "global config"** scope (settings.json edits, new hooks, statusline changes, any preference that takes effect across all projects), after the real change lands, **also update every related global-config file** so they stay in sync with the actual config. Related files include:

- `~/.claude/my-global-config-prompt.md` — portable replication prompt
- `~/.claude/CLAUDE.md` — this file, when the change is a new behavioral preference that should auto-apply
- Hook scripts, statusline scripts, or other artifacts referenced from `settings.json`

**Why:** These files mirror each other. Any drift means the next replication on a new machine, or the next session's auto-applied behavior, will quietly disagree with the real config. The user shouldn't have to hand-reconcile them.

**How to apply:**
- Triggers: phrases like 用户全局 / 全局配置 / 全局 settings / 所有项目生效, when an actual change has landed.
- Scope: update only when intent changes (add/remove/rewording). Pure parameter tweaks that don't change intent — skip.
- Decide which related files apply per change:
  - settings.json field flip → `my-global-config-prompt.md`
  - New behavioral preference / cross-project rule → `CLAUDE.md` **and** `my-global-config-prompt.md`
  - Hook/statusline script edit → that script + `my-global-config-prompt.md`'s description of it
- Style for `my-global-config-prompt.md`: keep the existing Chinese prose and sectioned layout. Field names and filenames (`settings.json`, `MEMORY.md`, `hooks.SessionEnd`, etc.) are allowed; **absolute paths are not** — use relative descriptions like "用户级 .claude/ 下".
- After updating, report the path(s) per the **Always report output paths** rule above.

**Then sync to the GitHub backup repo:**
- Repo: `https://github.com/wusss1412/config-prompt`, working copy at `D:\workspace\config-prompt`.
- **Scope: markdown only.** The backup repo mirrors `CLAUDE.md` and `my-global-config-prompt.md` — these are the *source of truth for intent*. Scripts referenced from `settings.json` (statusline `.ps1`, hooks `.js`, etc.) stay **local-only** and are regenerated from the prompt on a fresh machine; do not copy them into the repo. So when a hook/statusline script changes, the repo commit only carries the markdown updates that describe the new behavior.
- **Local steps run automatically (no confirmation):** copy updated files into `D:\workspace\config-prompt\claude-code\` (overwriting; other subdirs like `codex/` are off-limits), then `git add claude-code/<file> ...` + `git commit -m "..."`. Chain these in a single Bash invocation to avoid index races with the user's parallel edits in the same repo.
- Only stage paths under `claude-code/`. Never `git add .` / `git add -A` — the user's WIP in other subdirs must not be swept in.
- **`git push` requires confirmation per the Network operations rule above.** Surface the staged commit (hash + message + file list) and wait for explicit go-ahead before pushing. The standing authorization for this repo selects the *destination*; it does not waive the push-time confirmation.
- Force-push and other destructive git ops always require separate, explicit confirmation regardless.
