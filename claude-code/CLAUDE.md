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
