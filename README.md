Note: If you want a more autonomous setup for agentic workflows, check out [klaudworks/ralph-meets-rex](https://github.com/klaudworks/ralph-meets-rex).

> **Fork note:** This is a fork of [skills-directory/skill-codex](https://github.com/skills-directory/skill-codex) with an added fix for containerized environments (dev containers, Docker) where AppArmor blocks bwrap user namespace creation. See the [Container/Sandbox Bypass](#containersandbox-bypass) section below.

# Codex Integration for Claude Code

<img width="2288" height="808" alt="skillcodex" src="https://github.com/user-attachments/assets/85336a9f-4680-479e-b3fe-d6a68cadc051" />


## Purpose
Enable Claude Code to invoke the Codex CLI (`codex exec` and session resumes) for automated code analysis, refactoring, and editing workflows.

## Prerequisites
- `codex` CLI installed and available on `PATH`.
- Codex configured with valid credentials and settings.
- Confirm the installation by running `codex --version`; resolve any errors before using the skill.

## Installation

This repository is structured as a [Claude Code Plugin](https://code.claude.com/docs/en/plugins) with a marketplace. You can install it as a **plugin** (recommended) or extract it as a **standalone skill**.

### Option 1: Plugin Installation (Recommended)

Install via Claude Code's plugin system for automatic updates:

```
/plugin marketplace add guchengwei/skill-codex
/plugin install skill-codex@skill-codex
```

Or add to your `~/.claude/settings.json`:

```json
"extraKnownMarketplaces": {
  "skill-codex": {
    "source": {
      "source": "github",
      "repo": "guchengwei/skill-codex"
    }
  }
}
```

### Option 2: Standalone Skill Installation

Extract the skill folder manually:

```
git clone --depth 1 https://github.com/guchengwei/skill-codex.git /tmp/skills-temp && \
mkdir -p ~/.claude/skills && \
cp -r /tmp/skills-temp/plugins/skill-codex/skills/codex ~/.claude/skills/codex && \
rm -rf /tmp/skills-temp
```

## Container/Sandbox Bypass

If you're running inside a dev container or Docker where the host kernel has `apparmor_restrict_unprivileged_userns=1`, bwrap will fail with:

```
No permissions to create a new namespace
```

**Fix:** use `--dangerously-bypass-approvals-and-sandbox` instead of `--full-auto`:

```bash
codex exec -m <model> -s danger-full-access --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check -C <dir> "prompt" 2>/dev/null
```

> Note: `--dangerously-bypass-approvals-and-sandbox` is mutually exclusive with `--full-auto`. This is safe when you're already inside an externally-sandboxed container.

## Usage

### Important: Thinking Tokens
By default, this skill suppresses thinking tokens (stderr output) using `2>/dev/null` to avoid bloating Claude Code's context window. If you want to see the thinking tokens for debugging or insight into Codex's reasoning process, explicitly ask Claude to show them.

### Example Workflow

**User prompt:**
```
Use codex to analyze this repository and suggest improvements for my claude code skill.
```

**Claude Code response:**
Claude will activate the Codex skill and:
1. Ask which model to use (`gpt-5.4`, `gpt-5.3-codex-spark`, or `gpt-5.3-codex`) unless already specified in your prompt.
2. Ask which reasoning effort level (`low`, `medium`, or `high`) unless already specified in your prompt.
3. Select appropriate sandbox mode (defaults to `read-only` for analysis)
4. Run a command like:
```bash
codex exec -m gpt-5.3-codex-spark \
  --config model_reasoning_effort="high" \
  --sandbox read-only \
  --full-auto \
  --skip-git-repo-check \
  "Analyze this Claude Code skill repository comprehensively..." 2>/dev/null
```

**Result:**
Claude will summarize the Codex analysis output, highlighting key suggestions and asking if you'd like to continue with follow-up actions.

### Detailed Instructions
See [`plugins/skill-codex/skills/codex/SKILL.md`](plugins/skill-codex/skills/codex/SKILL.md) for complete operational instructions, CLI options, and workflow guidance.

## License

MIT License - see [LICENSE](LICENSE) for details.
