---
name: doc-changes
description: Document changes in a repository by updating the appropriate README(s) and agent documentation files. Detects what changed via git diff, targets the most contextually relevant README(s), updates any discovered agent docs (AGENTS.md, CLAUDE.md, .github/copilot-instructions.md, .cursorrules, .windsurfrules, GEMINI.md), flags changes not made by Claude for user clarification, and writes updates directly with a summary. Use when user wants to document recent changes, update docs after a task, or mentions "/doc-changes".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Document Changes

Document all recent changes by updating the appropriate README(s) and agent documentation files.

## Invocation

```
/doc-changes [git-ref]
```

- No argument: diff against `HEAD` (documents uncommitted changes)
- With ref: `/doc-changes main` diffs against that ref (documents all changes since branching)

## Workflow

### Step 1 - Detect Changes

**If a git repo exists:**

```bash
# Get the diff base (use argument if provided, else HEAD)
git diff <ref-or-HEAD> --name-only
git diff <ref-or-HEAD> --stat

# Also capture untracked files (git diff misses these)
git ls-files --others --exclude-standard
```

Collect the full list of changed files by **merging both outputs** - `git diff` covers modified/deleted tracked files, and `git ls-files --others` covers new untracked files. Together these form the authoritative change set.

**If no git repo exists:**

Fall back to Claude's session context - use the list of files edited via tool calls in this conversation. Note the limitation to the user: "No git repository detected. Documenting based on files edited in this session only."

### Step 2 - Flag External Changes

Compare the git diff file list against the files Claude edited via tool calls in the current session.

For any file that appears in the git diff but was **not** edited by Claude in this session, pause and ask:

> "I see changes in `<path>` that I didn't make. Can you give me context on what changed there so I can document it accurately?"

Wait for the user's response before continuing. Incorporate their explanation into the documentation for that file.

### Step 3 - Discover Documentation Files

Scan the repository recursively for:

**Human-facing docs:**
- `README.md` (root and all subdirectories)

**Agent docs (all formats, all locations):**
- `AGENTS.md`
- `CLAUDE.md`
- `.github/copilot-instructions.md`
- `.cursorrules`
- `.cursor/rules/*.mdc`
- `.windsurfrules`
- `GEMINI.md`

```bash
# Find all READMEs
find . -name "README.md" -not -path "*/node_modules/*" -not -path "*/.git/*"

# Find all agent docs
find . \( -name "AGENTS.md" -o -name "CLAUDE.md" -o -name "copilot-instructions.md" -o -name ".cursorrules" -o -name ".windsurfrules" -o -name "GEMINI.md" \) -not -path "*/node_modules/*" -not -path "*/.git/*"
```

**If no README exists anywhere:** Ask the user:
> "No README found in this repository. Would you like me to create one?"

If yes, generate a root `README.md` based on the repo structure and contents before continuing.

### Step 4 - Select Target README(s)

For each changed file, find the nearest `README.md` by walking up the directory tree. Group changed files by their nearest README.

Then apply judgment:
- If many files changed across many different subdirectories, consolidate to the root README rather than updating a large number of subfolder READMEs.
- If changes are tightly scoped to one subdirectory that has its own README, update that subfolder README primarily, and add a high-level note to the root README.
- If changes are significant and cross-cutting, update the root README as the primary target.
- The root README always receives at minimum a brief high-level summary of significant changes.

**Never create a new subfolder README** unless you are confident it is warranted by the scope and permanence of the changes in that directory. **Always ask the user before creating a new subfolder README.**

### Step 5 - Update README(s)

For each target README, read its current content first, then apply surgical updates:

**Content principles (human-facing):**
- Write from the perspective of a developer reading this for the first time.
- Describe *what changed and why it matters*, not implementation details.
- Use plain language. No internal file paths unless describing structural changes.
- Find the most semantically appropriate existing section and update it.
- Create a new section only if nothing in the existing structure fits.
- **Never remove or significantly rewrite existing content** without asking the user first.
- Do not add a `## Changelog` section unless one already exists or the user specifically wants one.

**Full rewrite mode:** Only if the user passed `--rewrite` as an argument. Otherwise, surgical updates only.

### Step 6 - Update Agent Docs

For each discovered agent doc, read it and determine whether the changes are relevant to its audience (an AI agent working in this repo).

Update agent docs with technical/structural information:
- What new files or modules were added and what they do
- Conventions or patterns introduced by the changes
- Anything an AI agent must know to work correctly in this repo going forward
- Breaking changes to interfaces, APIs, or file structures

**Content principles (AI-facing):**
- Be precise and technical. File paths, function names, and patterns are appropriate here.
- Focus on what an AI agent needs to *know* or *avoid* when working in this codebase.
- Do not duplicate what is already clearly documented elsewhere in the file.
- Apply the same surgical edit rule: never remove existing guidance without asking.

### Step 7 - Write and Summarize

Write all updates directly without preview (changes are reversible via git).

**Exception:** If no git repo exists, show a preview of each proposed change and ask for confirmation before writing.

After all writes are complete, print a concise summary:

```
Documentation updated:

README(s):
  - README.md - added X to "Features" section
  - src/api/README.md - updated "Endpoints" section

Agent docs:
  - CLAUDE.md - added note about new module structure
  - .github/copilot-instructions.md - updated conventions

Skipped (no relevant changes):
  - AGENTS.md
```

## Edge Cases

- **Clean working tree / no changes detected:** Only after checking both `git diff` (tracked changes) and `git ls-files --others` (untracked files) return nothing, inform the user: "No changes detected against `<ref>`. Nothing to document."
- **Binary files in diff:** Skip them for documentation purposes unless their presence is structurally significant (e.g. a new asset directory).
- **Changes only to documentation files:** Skip (no need to document documentation changes).
- **Changes only to lock files or generated files** (e.g. `package-lock.json`, `*.generated.ts`): Skip unless the underlying dependency change is worth noting.
