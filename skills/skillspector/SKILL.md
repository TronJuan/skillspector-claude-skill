---
name: skillspector
description: >
  MUST USE before installing, trusting, or recommending any AI agent skill,
  Claude Code skill, plugin, MCP server, or LangGraph agent from an external
  source (a GitHub repo, a zip, a URL, or a shared skill folder). Scans for
  risky instructions, hidden metadata, prompt injection, data exfiltration,
  privilege escalation, and supply-chain issues before the content is ever
  loaded or run. Trigger on requests like "scan this skill", "is this safe
  to install", "check this MCP server for risks", "audit this agent skill".
metadata:
  homepage: https://github.com/TronJuan/skillspector-claude-skill
  scanner: https://github.com/TronJuan/SkillSpector/tree/fp-reduction
---

# SkillSpector — Agent Skill Security Scanner

Runs [NVIDIA's SkillSpector](https://github.com/NVIDIA/SkillSpector) from the
`fp-reduction` branch of [TronJuan/SkillSpector](https://github.com/TronJuan/SkillSpector/tree/fp-reduction).
That branch adds contextual scoring, which cuts false "DO NOT INSTALL"
verdicts on legitimate skills without weakening detection of malicious ones.

## First use: install the scanner

Check whether it is installed:

```bash
skillspector --help
```

If the command is not found, install it with [uv](https://docs.astral.sh/uv/)
(needs Python 3.12–3.14; uv downloads one if needed):

```bash
uv tool install "git+https://github.com/TronJuan/SkillSpector@fp-reduction"
```

If uv is missing, ask the user before installing it (`pip install uv`, or the
installer at https://docs.astral.sh/uv/getting-started/installation/). After
installing, open a new shell if `skillspector` is still not on PATH, or run
`uv tool update-shell`.

## Running a scan

```bash
SKILLSPECTOR_SCORING=contextual skillspector scan <INPUT_PATH> --no-llm
```

On Windows PowerShell, set the variable first:
`$env:SKILLSPECTOR_SCORING = "contextual"; skillspector scan <INPUT_PATH> --no-llm`

`<INPUT_PATH>` accepts a local directory or file, a git repo URL, or a zip file.

- Default to `--no-llm` (fast static analysis, no API key, covers the known
  vulnerability patterns).
- Always set `SKILLSPECTOR_SCORING=contextual`. It weights findings by file
  role (SKILL.md and scripts count fully; tests, examples and READMEs count
  less), stops large repos from saturating at 100, and reports
  dependency-hygiene findings (SC1, SC4, SC5, LP3, LP4) without letting them
  drive the verdict. Mention any hygiene findings separately.
- For deeper semantic analysis that filters false positives, drop `--no-llm`
  and set `SKILLSPECTOR_PROVIDER=claude_cli`. It uses the existing Claude Code
  login, so no API key is needed, but skill file contents are sent to the
  model. Ask the user first, and offer it when a static scan returns
  HIGH/CRITICAL on a reputable repo.
- Add `--recursive` when scanning a directory that holds several skills, so
  each subdirectory with a `SKILL.md` is scanned on its own.
- Add `--format json -o report.json` when the user wants a machine-readable
  report to save or compare later.

## Reading the report

The report gives a 0–100 risk score, a severity band (LOW/MEDIUM/HIGH/
CRITICAL), and a recommendation (SAFE / CAUTION / DO NOT INSTALL), followed
by findings with file:line location, confidence and a suggested fix.

- Lead with the score, severity and recommendation: that answers "is this
  safe to install".
- List findings above LOW severity; skip trivial ones unless asked for the
  full report.
- If `Coverage` is under 100% or `Status` is not `successful`, say so. An
  incomplete scan is not a clean bill of health.
- Static analysis can miss things and can flag legitimate code. Treat the
  verdict as evidence for the user's decision, not a guarantee.
- Never install or run a skill that scores HIGH/CRITICAL or gets a DO NOT
  INSTALL recommendation without flagging it to the user and getting their
  go-ahead first.

## Updating the scanner

```bash
uv tool upgrade skillspector
```
