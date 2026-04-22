# Cursor skills: coding-guide & safe-coding

Two [Agent Skills](https://docs.cursor.com) for Cursor (or compatible) use.

| Directory        | `name`         | Role |
|-----------------|----------------|------|
| `coding-guide/` | `coding-guide` | Base behavior: simplicity, surgical edits, assumptions, verifiable goals. `alwaysApply: true` in the published `SKILL.md`. |
| `safe-coding/`  | `safe-coding`  | File/encoding/imports/secrets and optional security nudges. Invoked as required by `coding-guide`. |

## Install (personal)

Copy each folder into your user skills directory so the structure is:

```text
~/.cursor/skills/coding-guide/SKILL.md
~/.cursor/skills/safe-coding/SKILL.md
```

On Windows, `~` is usually `C:\Users\<you>\`.

## Install (project)

Copy the two folders into the project:

```text
<repo>/.cursor/skills/coding-guide/SKILL.md
<repo>/.cursor/skills/safe-coding/SKILL.md
```

## License

No license set by default; add a `LICENSE` file in this repo if you need one.
