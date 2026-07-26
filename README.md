# Fungi skills

Skills for installing and operating [Fungi](https://github.com/enbop/fungi), a private multi-device service platform.

The repository uses the standard `SKILL.md` format and can be installed into any agent supported by the [`skills` CLI](https://github.com/vercel-labs/skills#supported-agents), including Claude Code, Codex, Cursor, Gemini CLI, OpenCode, and many others.

## Install

List the skills available in this repository:

```bash
npx skills@latest add enbop/fungi-skills --list
```

Install the Fungi skill and let the CLI detect or prompt for available agents:

```bash
npx skills@latest add enbop/fungi-skills --skill fungi
```

Install it globally instead of in the current project:

```bash
npx skills@latest add enbop/fungi-skills --skill fungi --global
```

Optionally target one or more agents explicitly using their supported agent IDs:

```bash
npx skills@latest add enbop/fungi-skills --skill fungi --agent AGENT_ID
```

Product-specific metadata under `agents/` is optional. The Fungi instructions and references remain portable across compatible agents.

## Included skills

- `fungi`: Install and initialize Fungi, connect devices with explicit trust approval, manage local and remote services, apply recipes, create `.fungi.md` service files, and diagnose results with inspect and bounded logs.

## Feedback

Found a problem or have a suggestion?

- Fungi CLI, daemon, or service behavior: [enbop/fungi issues](https://github.com/enbop/fungi/issues)
- Fungi App behavior: [enbop/fungi-app issues](https://github.com/enbop/fungi-app/issues)
- This skill's instructions: [enbop/fungi-skills issues](https://github.com/enbop/fungi-skills/issues)
