# AI Agent Instructions: Code Audit Pack

You are an AI coding agent. Your task is to install/read this Code Audit Pack and use it as the source of truth before auditing or modifying any repository.

This pack is agent-agnostic. It works for Claude Code, Codex CLI, Hermes Agent, Cursor/Continue, Pi/coding agents, or any autonomous coding agent that can read files and run shell commands.

## Required Files

Do not use only `SKILL.md`. The full audit pack requires all files below:

```text
code-audit-skill/README.md
code-audit-skill/SKILL.md
code-audit-skill/PROMPT.md
code-audit-skill/references/audit-dimensions.md
code-audit-skill/references/fix-patterns.md
code-audit-skill/references/severity-guide.md
```

If any file is missing or empty, stop and fetch it before continuing.

## Quick Install for Any AI Agent

Run this from the workspace where you will audit code:

```bash
mkdir -p code-audit-skill/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"

curl -fsSL "$BASE/README.md" -o code-audit-skill/README.md
curl -fsSL "$BASE/SKILL.md" -o code-audit-skill/SKILL.md
curl -fsSL "$BASE/PROMPT.md" -o code-audit-skill/PROMPT.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o code-audit-skill/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o code-audit-skill/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o code-audit-skill/references/severity-guide.md

test -s code-audit-skill/README.md
test -s code-audit-skill/SKILL.md
test -s code-audit-skill/PROMPT.md
test -s code-audit-skill/references/audit-dimensions.md
test -s code-audit-skill/references/fix-patterns.md
test -s code-audit-skill/references/severity-guide.md
```

After all `test -s` commands pass, read the files in this order:

1. `code-audit-skill/PROMPT.md`
2. `code-audit-skill/references/audit-dimensions.md`
3. `code-audit-skill/references/severity-guide.md`
4. `code-audit-skill/references/fix-patterns.md`
5. `code-audit-skill/SKILL.md` if you need the Hermes-specific trigger/SOP wrapper

Then execute the audit exactly as instructed in `PROMPT.md`.

## Copy-Paste Instruction for Another AI Agent

Use this block when instructing Claude Code, Codex CLI, Pi, Cursor, or another coding agent:

```text
Install and use the Code Audit Pack before auditing or editing code.

Run:
mkdir -p code-audit-skill/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"
curl -fsSL "$BASE/README.md" -o code-audit-skill/README.md
curl -fsSL "$BASE/SKILL.md" -o code-audit-skill/SKILL.md
curl -fsSL "$BASE/PROMPT.md" -o code-audit-skill/PROMPT.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o code-audit-skill/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o code-audit-skill/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o code-audit-skill/references/severity-guide.md

test -s code-audit-skill/README.md && \
test -s code-audit-skill/SKILL.md && \
test -s code-audit-skill/PROMPT.md && \
test -s code-audit-skill/references/audit-dimensions.md && \
test -s code-audit-skill/references/fix-patterns.md && \
test -s code-audit-skill/references/severity-guide.md

Then read `code-audit-skill/PROMPT.md` fully and read every file in `code-audit-skill/references/` fully.
Treat those files as the source of truth for the audit.
Do not continue if any required file is missing.
Do not fabricate audit findings, command output, or verification results.
Audit the target repository against the available PRD/spec/reference code. If none exists, use self-discovery mode from the prompt.
Fix every finding that the prompt classifies as requiring a fix.
Run the repository's real verification suite.
Re-audit after fixes.
Do not claim completion until the prompt's completion criteria are satisfied with real command output.
```

## Clone Instead of curl

If curl is unavailable or you prefer cloning:

```bash
git clone https://github.com/rickicode/My-Skills.git /tmp/my-skills
cp -a /tmp/my-skills/skills/software-development/code-audit ./code-audit-skill

test -s code-audit-skill/README.md
test -s code-audit-skill/SKILL.md
test -s code-audit-skill/PROMPT.md
test -s code-audit-skill/references/audit-dimensions.md
test -s code-audit-skill/references/fix-patterns.md
test -s code-audit-skill/references/severity-guide.md
```

Then read `./code-audit-skill/PROMPT.md` and all `./code-audit-skill/references/*.md` before auditing.

## Offline / Manual Save Mode

If network access is blocked, the user must provide and save all files exactly like this:

```text
code-audit-skill/README.md
code-audit-skill/SKILL.md
code-audit-skill/PROMPT.md
code-audit-skill/references/audit-dimensions.md
code-audit-skill/references/fix-patterns.md
code-audit-skill/references/severity-guide.md
```

Once saved, verify every file is non-empty, then read `PROMPT.md` and all references before work.

## Optional: Install as a Hermes Skill

Only use this section if the runtime is Hermes Agent and you want the skill discoverable via `skill_view` / skill loading.

```bash
mkdir -p ~/.hermes/skills/software-development/code-audit/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"

curl -fsSL "$BASE/SKILL.md" -o ~/.hermes/skills/software-development/code-audit/SKILL.md
curl -fsSL "$BASE/PROMPT.md" -o ~/.hermes/skills/software-development/code-audit/PROMPT.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o ~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o ~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o ~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Start a fresh Hermes session after installing so the skill loader can detect it.

## Operating Rules for the Agent

- Always read `PROMPT.md` and all `references/*.md` before starting the audit.
- Never audit from memory only.
- Never skip required files because they seem redundant.
- Never claim tests/build/lint passed unless you actually ran them and saw exit code `0`.
- If the target repo has PRD/spec/reference code, use it as audit truth.
- If no PRD/spec/reference exists, use self-discovery mode from `PROMPT.md`.
- If fixes are applied, run verification and re-audit after fixes.
- If any requirement is unclear, ask for the minimum missing information only.
