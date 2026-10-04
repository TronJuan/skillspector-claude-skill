# SkillSpector for Claude Code

A Claude Code skill that scans AI agent skills, plugins and MCP servers for security risks **before** you install them: prompt injection, credential exfiltration, persistence, obfuscated code and supply-chain issues.

It runs [NVIDIA's SkillSpector](https://github.com/NVIDIA/SkillSpector) from my [`fp-reduction` branch](https://github.com/TronJuan/SkillSpector/tree/fp-reduction), which adds contextual scoring. On 823 non-malicious skills, false "DO NOT INSTALL" verdicts dropped 66% (266 → 91), while all 11 malicious test samples were still flagged. Those numbers come from static analysis only.

## Install

In Claude Code:

```
/plugin marketplace add TronJuan/skillspector-claude-skill
/plugin install skillspector@skillspector
```

Then just ask, for example *"is this skill safe to install? https://github.com/someone/some-skill"*. The first time, Claude installs the scanner for you with [uv](https://docs.astral.sh/uv/) (Python 3.12–3.14):

```bash
uv tool install "git+https://github.com/TronJuan/SkillSpector@fp-reduction"
```

### Manual install (no plugin)

Copy `skills/skillspector/` into `~/.claude/skills/`, then run the `uv tool install` command above.

## Use the scanner on its own

```bash
SKILLSPECTOR_SCORING=contextual skillspector scan <path-or-repo-url> --no-llm
```

You get a 0–100 risk score, a severity band and a SAFE / CAUTION / DO NOT INSTALL recommendation, with each finding's file and line.

## What the fp-reduction branch changes

| Change | Problem it fixes |
|---|---|
| The credential-flow rule checks where a key is sent | An API key sent to its own vendor's domain (e.g. `FIRECRAWL_API_KEY` to `api.firecrawl.dev`) was rated CRITICAL "credential exfiltration" |
| Control arguments are not treated as data | `verify=not os.environ.get(...)` was rated CRITICAL |
| App state is separated from host persistence | A skill saving its own notes file was flagged as "session persistence" |
| Opt-in contextual scoring (`SKILLSPECTOR_SCORING=contextual`) | Large repos always scored 100; README text counted like `SKILL.md`; outdated dependencies decided the verdict |

The legacy score is unchanged unless you opt in, and the upstream test suite still passes. Evasion guards (look-alike subdomains, `@` in URLs, mixed-secret payloads) each have tests.

## Limits

- Static analysis can miss things and can flag legitimate code. Treat the verdict as evidence, not a guarantee.
- The malicious benchmark is small (11 samples), and some blocking margins are thin.

## Credit and licence

SkillSpector is NVIDIA's project, licensed under Apache 2.0. This repo packages it as a Claude Code skill and points to my branch of it. It is not an official NVIDIA release. This repo is also Apache 2.0.
