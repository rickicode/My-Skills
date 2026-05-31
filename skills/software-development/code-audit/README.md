# Code Audit Skill / Agent Instruction Pack

Portable audit instruction pack untuk coding agent apa pun: Hermes Agent, Claude Code, Codex CLI, Cursor/Continue, Pi/coding agent lain, atau AI agent custom.

Tujuannya: agent bisa melakukan audit kode, validasi production-readiness, compliance terhadap PRD/reference code, lalu autofix sampai hasilnya benar-benar terverifikasi.

## Isi Paket

Paket ini bukan hanya `SKILL.md`. Semua file berikut dibutuhkan agar audit lengkap:

- `SKILL.md` — versi Hermes skill, juga bisa dibaca agent lain sebagai SOP utama.
- `PROMPT.md` — prompt portable yang bisa langsung diberikan ke Claude/Codex/agent lain.
- `references/audit-dimensions.md` — dimensi audit yang harus dicek.
- `references/fix-patterns.md` — pola fix umum dan cara eksekusinya.
- `references/severity-guide.md` — panduan severity untuk temuan audit.

## Pakai Langsung Tanpa Hermes

Untuk Claude Code, Codex CLI, Pi/coding agent, atau agent lain, cukup download semua file ke folder kerja lalu suruh agent membaca `PROMPT.md` dan semua `references/*.md` sebelum audit.

```bash
mkdir -p code-audit-skill/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"

curl -fsSL "$BASE/PROMPT.md" -o code-audit-skill/PROMPT.md
curl -fsSL "$BASE/SKILL.md" -o code-audit-skill/SKILL.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o code-audit-skill/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o code-audit-skill/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o code-audit-skill/references/severity-guide.md
```

Lalu berikan instruksi ini ke agent:

```text
Read `code-audit-skill/PROMPT.md` completely.
Then read every file in `code-audit-skill/references/` completely.
Use those files as the audit SOP and execute the audit/autofix against the target repository.
Do not continue if any required file is missing.
Do not declare completion until the verification and re-audit requirements in the prompt are satisfied.
```

## One-Shot Prompt untuk Agent Lain

Copy-paste instruksi ini ke Claude Code / Codex / Pi / coding agent lain:

```text
Before doing any code audit or fix, download the full Code Audit instruction pack:

mkdir -p code-audit-skill/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"
curl -fsSL "$BASE/PROMPT.md" -o code-audit-skill/PROMPT.md
curl -fsSL "$BASE/SKILL.md" -o code-audit-skill/SKILL.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o code-audit-skill/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o code-audit-skill/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o code-audit-skill/references/severity-guide.md

Then:
1. Verify all files exist and are non-empty.
2. Read `code-audit-skill/PROMPT.md` fully.
3. Read all files under `code-audit-skill/references/` fully.
4. Use those instructions as the source of truth for the audit/autofix.
5. Audit the target repository against the available PRD/spec/reference code. If none exists, use self-discovery mode.
6. Fix every finding that the prompt classifies as requiring a fix.
7. Run the full verification suite for the repository.
8. Re-audit after fixes. Do not claim completion until there are no unresolved findings and verification passes.
```

## Install sebagai Hermes Skill

Kalau environment-nya Hermes Agent, install ke folder skill Hermes:

```bash
mkdir -p ~/.hermes/skills/software-development/code-audit/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"

curl -fsSL "$BASE/SKILL.md" -o ~/.hermes/skills/software-development/code-audit/SKILL.md
curl -fsSL "$BASE/PROMPT.md" -o ~/.hermes/skills/software-development/code-audit/PROMPT.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o ~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o ~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o ~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Buka session Hermes baru setelah install supaya skill loader membaca skill baru.

## Install via Clone

Untuk Hermes:

```bash
git clone https://github.com/rickicode/My-Skills.git /tmp/my-skills
mkdir -p ~/.hermes/skills/software-development
cp -a /tmp/my-skills/skills/software-development/code-audit ~/.hermes/skills/software-development/
```

Untuk agent non-Hermes:

```bash
git clone https://github.com/rickicode/My-Skills.git /tmp/my-skills
cp -a /tmp/my-skills/skills/software-development/code-audit ./code-audit-skill
```

Lalu instruksikan agent membaca:

```text
Read ./code-audit-skill/PROMPT.md and every file under ./code-audit-skill/references/ before auditing or editing code.
```

## Simpan Semua File yang Dibutuhkan

Jika environment tidak boleh network, copy semua file ini ke folder yang sama:

```text
code-audit-skill/SKILL.md
code-audit-skill/PROMPT.md
code-audit-skill/references/audit-dimensions.md
code-audit-skill/references/fix-patterns.md
code-audit-skill/references/severity-guide.md
```

Untuk Hermes, path-nya:

```text
~/.hermes/skills/software-development/code-audit/SKILL.md
~/.hermes/skills/software-development/code-audit/PROMPT.md
~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Jangan hanya menyimpan `SKILL.md`; agent butuh `PROMPT.md` dan semua referensi agar audit tidak dangkal.

## Verifikasi Download

Untuk folder portable:

```bash
test -s code-audit-skill/SKILL.md
test -s code-audit-skill/PROMPT.md
test -s code-audit-skill/references/audit-dimensions.md
test -s code-audit-skill/references/fix-patterns.md
test -s code-audit-skill/references/severity-guide.md
```

Untuk Hermes:

```bash
test -s ~/.hermes/skills/software-development/code-audit/SKILL.md
test -s ~/.hermes/skills/software-development/code-audit/PROMPT.md
test -s ~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
test -s ~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
test -s ~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Jika semua command exit `0`, paket audit lengkap sudah tersimpan.
