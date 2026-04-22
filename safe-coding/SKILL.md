---
name: safe-coding
description: Edits source files safely to avoid garbled text, UTF-8 BOM issues, import mistakes, and hardcoded secrets or credential logging across Java, TypeScript/JavaScript, Vue/SFC, styles, and config. Surfaces optional security reminders (validation, injection, XSS, paths, authz) without forcing refactors unless asked. Use when modifying backend or frontend code, imports, secrets-related changes, comments, or encoding-sensitive content.
---

# Safe Coding

Covers encoding, imports/modules, and low-risk edits for **both backend and frontend**. This skill does not cover general project style or process; use other project skills or rules for that.

## Scope

Apply when touching typical project sources, for example:

- **Backend:** Java (`service` / `controller` / `mapper` / `entity` / `dto`), XML/resources if edited as text
- **Frontend:** `.vue`, `.tsx`, `.jsx`, `.ts`, `.js`, `.css`, `.scss`, `.html`
- **Config / data:** `.json`, `.yaml`, `.yml` (as text)

## Hard Guardrails (Do Not Skip)

### 1) Fresh content before write (decision flow)

Goal: patch against content that matches disk. Skip redundant reads only when staleness is ruled out.

**Must re-read (or equivalent fresh snapshot) before applying a patch when any is true:**

- First time you touch this file **in this assistant turn** for the current edit.
- **Human may have changed** the file since your last read: user said they edited it; merge with their edits; file had **unsaved changes** you did not fold in; or you are not the sole writer.
- Another **tool step** may have modified the file (script, formatter, codegen, git, merge).
- Last patch **failed**, was **partial**, or the diff was **unexpected** → re-read before retry.
- You will write **again** to the same file **after** other steps, other files, or a **long pause** → treat prior read as possibly stale.

**Skip re-read only when all hold:**

- You **already read** this file in **this assistant turn** (or have an equally fresh full snapshot).
- **No intervening edit** could have landed; **user did not** indicate edits since that read.
- Next step is **immediately** applying the patch from that snapshot.

**Manual edits in the IDE:** If the user could have saved after your last read, default to **re-read**.

### 2) Import / module policy

- Add `import` / `require` / `import()` / dynamic imports **only when required** by the change.
- **Removing imports:** You **may delete** imports (or duplicate/unused module lines) **after you confirm they are unused** — e.g. TypeScript/Java “unused import” diagnostics, build output, linter, plus **identifiers used** in the file (not grep alone — see edge cases below). Do **not** remove imports you only “think” are unused without that check.
- **Do not** mass-reorder or reformat import blocks for style unless the user asks (minimize churn).
- **Java:** Keep `package` first, then `import` block; removing lines is OK when unused is verified.
- **Vue SFC `<script>`:** Same as JS/TS — add as needed; remove when verified unused.

**Edge cases before deleting an import (do not rely on naive text search alone):**

- **Side-effect imports:** `import './polyfill'` or `import 'some-package/register'` may have **no referenced symbol** but still required at load time — do not remove unless the user confirms or bundler/build proves it unused.
- **`import type` / type-only:** Used only in types; ensure TS analysis/linter marks unused before removing.
- **Compiler / macro magic:** Vue compiler macros, re-exports, codegen — if unsure, keep the import or verify with project tooling.
- **Default:** Prefer **IDE/linter/compiler “unused import”** over deleting because the identifier string does not appear.

### 3) Encoding safety

- **Hand-edited text sources** (Java, TS/JS, Vue, CSS, JSON/YAML edited as text, etc.) should remain **UTF-8 without BOM**.
- Do not introduce a leading `\ufeff` or save as “UTF-8 with BOM”.
- Avoid workflows that silently change encoding.
- **Binary vs text:** Do **not** patch binary assets (images, fonts, jars, compiled blobs) as if they were UTF-8 source; use appropriate tools or ask the user.
- **Non-UTF-8 legacy resources:** If the project intentionally uses another encoding for a given file (some `*.properties`, legacy bundles), **do not** blindly convert to UTF-8 unless the task explicitly asks — avoid corrupting meaning or breaking tooling.

### 4) Chinese text (anti-garbled)

- Preserve existing Chinese in comments and string literals unless the task requires changes.
- If content is already corrupted, fix only what is requested unless the user wants a full-file cleanup.
- After edits, ensure you did not introduce new garbled text.

### 5) Sensitive information, secrets, and keys (hard)

- **Do not** add or ask the user to add real **passwords, API keys, tokens, private keys, or full connection strings** into source code, comments, or committed config examples. Use env vars, the project’s existing secret/config pattern, or obvious placeholders (e.g. `YOUR_API_KEY`, `******`).
- **Do not** add new `console.log` / `log.info` / similar that print full tokens, cookies, or credentials. If logging is required, log **redacted** or **hashed** values only, or a non-sensitive id.
- If the user’s request would embed a secret, **refuse the unsafe part** and suggest the project’s standard way to load secrets.
- **Do not** create or expand files that look like real `.env` with live secrets; use `.env.example` with fake values if a template is needed.

## Security reminders (not mandatory to change)

**Do not** refactor or “fix” existing code for these unless the user explicitly asks. **Do** mention the risk in a short note when you touch related code or the user’s change would make it worse.

- **Input validation and boundaries:** External input (query, body, path, header) may need type/range checks on the **server**; the browser is not a security boundary. Suggest when new or broad user input is introduced.
- **SQL / NoSQL / command injection:** Prefer parameterization / bound parameters; avoid string-concatenated SQL or shell commands with user input. Remind when you see or add string-built queries or `exec`/`ProcessBuilder`/`child_process` with untrusted data.
- **XSS and HTML output:** `v-html`, `innerHTML`, rich text, and unescaped user content are high risk. Suggest escaping, sanitization, or a vetted library when such patterns are added or extended.
- **Path and file access:** User-controlled paths can lead to directory traversal; suggest normalizing and constraining to an allowed root when building file paths from input.
- **AuthZ / id in URL:** When adding or changing operations on a resource id, consider whether the current user is allowed to act on **that** id (horizontal access). Remind, do not redesign the app.
- **Errors to the client:** Production responses should not leak stack traces, SQL, or internal paths. Remind if new `e.printStackTrace` to the response or similar is introduced.

## Examples (plain language)

Think of these as quick “if this / then that” patterns.

### When to re-read the file

- **Scenario:** You read `OrderService.java` at the start of the turn; the user then says “I manually fixed the same file, merge with mine.”  
  **Do:** Re-read before patching. **Don’t:** Patch from your first snapshot.

- **Scenario:** Read file → immediately apply one patch in the same step, nothing else touched the file, user didn’t edit.  
  **Do:** That read can be enough. **Don’t:** Re-read out of habit every time.

### Imports

- **OK to remove:** `import com.example.UnusedUtil;` — you searched the file and `UnusedUtil` never appears. Remove the line.

- **Not OK to remove:** “This import looks unused” but you didn’t grep or check the linter. Keep it until confirmed.

### Secrets and logs (hard rules)

- **Bad:** `String password = "MyP@ssw0rd";` in source.  
  **Better:** Load from config/env the project already uses; or a placeholder in samples only.

- **Bad:** `log.info("token={}", fullJwt);`  
  **Better:** `log.info("token present, len={}", fullJwt == null ? 0 : fullJwt.length());` or log only a correlation id.

- **Bad:** committing a filled `.env` with real DB passwords.  
  **Better:** `.env.example` with `DB_PASSWORD=changeme`.

### Encoding and Chinese

- **Bad:** Saving a `.java` file as “UTF-8 with BOM” / mystery first character that breaks compilation.  
  **Better:** UTF-8 **without** BOM.

- **Bad:** Comments turn into ``??????`` after an edit.  
  **Better:** Preserve existing Chinese unless you intentionally change that text.

### Optional security reminders (mention only — don’t refactor unless asked)

- **Scenario:** User adds `v-html="userBio"` where `userBio` comes from the API.  
  **Say:** “Rendering raw HTML from user-controlled data can be XSS; consider sanitizing or escaping unless content is trusted.” No forced rewrite unless they ask.

- **Scenario:** New endpoint concatenates user input into SQL string.  
  **Say:** “Parameterized queries reduce injection risk.” Same — remind, don’t reshape the whole DAO unless requested.

## Operational Checklist

- [ ] Re-read when staleness was possible; otherwise used fresh read-in-turn per §1
- [ ] Imports/modules valid; removals only after **confirmed** unused
- [ ] No gratuitous import reordering
- [ ] Encoding remains UTF-8 without BOM where applicable
- [ ] No newly introduced garbled Chinese text
- [ ] No new hardcoded secrets, full tokens, or live credentials in code or logs
