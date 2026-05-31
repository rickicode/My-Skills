# Code Audit Skill

Hermes Agent skill untuk audit kode, validasi production-readiness, dan autofix berdasarkan PRD, kode referensi, atau self-discovery dari codebase.

Skill ini bukan hanya `SKILL.md`. Ia membutuhkan file pendukung berikut agar instruksi audit lengkap:

- `SKILL.md`
- `PROMPT.md`
- `references/audit-dimensions.md`
- `references/fix-patterns.md`
- `references/severity-guide.md`

## Install Cepat via curl

Jalankan command ini di mesin yang punya Hermes Agent:

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

```bash
git clone https://github.com/rickicode/My-Skills.git /tmp/my-skills
mkdir -p ~/.hermes/skills/software-development
cp -a /tmp/my-skills/skills/software-development/code-audit ~/.hermes/skills/software-development/
```

Buka session Hermes baru setelah copy selesai.

## Instruksi untuk AI Agent

Kalau ingin menyuruh AI agent lain memasang skill ini, kirim instruksi ini:

```text
Install the Hermes Agent `code-audit` skill before starting.

Run:
mkdir -p ~/.hermes/skills/software-development/code-audit/references
BASE="https://raw.githubusercontent.com/rickicode/My-Skills/main/skills/software-development/code-audit"
curl -fsSL "$BASE/SKILL.md" -o ~/.hermes/skills/software-development/code-audit/SKILL.md
curl -fsSL "$BASE/PROMPT.md" -o ~/.hermes/skills/software-development/code-audit/PROMPT.md
curl -fsSL "$BASE/references/audit-dimensions.md" -o ~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
curl -fsSL "$BASE/references/fix-patterns.md" -o ~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
curl -fsSL "$BASE/references/severity-guide.md" -o ~/.hermes/skills/software-development/code-audit/references/severity-guide.md

Then start a fresh Hermes session and load/use the `code-audit` skill.
Do not continue if any curl command fails, because the skill depends on all files above.
```

## Simpan Semua File yang Dibutuhkan

Jika environment tidak boleh network, simpan semua file ini ke path yang sama:

```text
~/.hermes/skills/software-development/code-audit/SKILL.md
~/.hermes/skills/software-development/code-audit/PROMPT.md
~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Jangan hanya menyimpan `SKILL.md`; referensi audit, severity, dan fix pattern diperlukan agar audit lengkap.

## Verifikasi Install

```bash
test -s ~/.hermes/skills/software-development/code-audit/SKILL.md
test -s ~/.hermes/skills/software-development/code-audit/PROMPT.md
test -s ~/.hermes/skills/software-development/code-audit/references/audit-dimensions.md
test -s ~/.hermes/skills/software-development/code-audit/references/fix-patterns.md
test -s ~/.hermes/skills/software-development/code-audit/references/severity-guide.md
```

Jika semua command exit `0`, file skill lengkap sudah tersimpan.
