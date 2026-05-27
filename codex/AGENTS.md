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

## DOCX Editing Rules Learned From Citation-Rebuild Errors

These rules apply when editing Word `.docx` thesis or academic documents, especially when changing references, citations, formulas, figures, or cross-references.

- Treat `.docx` files as ZIP packages and inspect `word/document.xml` directly. Prefer Python `zipfile` plus `lxml.etree` for parsing and validation when available.
- For citation or reference-list changes, use the user's original `.docx` as the base document. Do not rebuild or reorder body paragraphs unless the user explicitly asks for body rewriting.
- Keep body edits minimal: replace only the reference-list region and update existing `REF _Ref...` field codes, visible citation numbers, and matching bookmark targets.
- Do not flatten, recreate, or rewrite formula paragraphs. Formula parameters may be stored as separate Word runs, embedded objects, or special formatting rather than plain text.
- Before declaring a DOCX edit complete, compare the output against the original file before the `参考文献` heading:
  - body paragraph order and text, ignoring only intentional citation-number changes;
  - paragraph properties (`w:pPr`);
  - counts of `w:object`, `w:drawing`, subscript/superscript formatting, tabs, breaks, and section properties.
- Validate citation integrity after edits:
  - every visible citation is superscript when the original format uses superscript citations;
  - every `REF _Ref...` target has a matching `w:bookmarkStart`;
  - every reference bookmark is used when intended;
  - visible citation numbers match the target reference items.
- If a DOCX output shows unexpected content movement, such as section text appearing under the wrong heading, stop and rebuild from the original document base instead of patching around the corrupted output.
- Do not rely on file size, visual spot checks, or successful ZIP opening as proof of correctness. Run structural XML checks and report the exact validation results.

### DOCX Mistake Log And Required Countermeasures

- Mistake: replacing an entire paragraph as plain text can flatten Word run-level formatting, including subscripts, superscripts, formula variables, field codes, and cross-reference formatting.
  Countermeasure: before editing a paragraph, inspect its `w:r` structure. If it contains multiple runs, `w:vertAlign`, `w:fldChar`, `w:instrText`, `w:object`, or math elements, do not replace it wholesale. Preserve the original runs and make only targeted text changes, or add new content as a separate paragraph cloned from a compatible style.
- Mistake: after deleting figures, leaving正文 references such as `如图4.6所示` creates broken internal logic even if the image object and caption are gone.
  Countermeasure: after any figure/table deletion or insertion, scan all `图X.X` and `表X.X` references in `word/document.xml`; verify every reference has a matching caption and every caption has the expected nearby image/table object.
- Mistake: inserting a new figure without immediately validating image relationships can leave broken media targets or mismatched captions.
  Countermeasure: validate `word/_rels/document.xml.rels`, `[Content_Types].xml`, and every `a:blip/@r:embed` target after insertion.
- Mistake: fixing one visible issue and stopping early can miss unrelated text discontinuities introduced by previous edits.
  Countermeasure: run a full-document text scan for figure/table references, obviously truncated sentences, and edited-section continuity before declaring completion.
- Mistake: saying a DOCX was edited with ZIP/XML is not enough; the editing strategy must also be conservative at the run level.
  Countermeasure: report both the parsing method and the preservation method, including whether edited paragraphs preserved original runs or were intentionally added as new paragraphs.
