# Fungi skills

Agent skills for installing and operating [Fungi](https://github.com/enbop/fungi), a private multi-device service platform.

## Install

List the skills available in this repository:

```bash
npx skills@latest add enbop/fungi-skills --list
```

Install the Fungi skill:

```bash
npx skills@latest add enbop/fungi-skills --skill fungi
```

To install it globally for Codex:

```bash
npx skills@latest add enbop/fungi-skills --skill fungi --agent codex --global
```

## Included skills

- `fungi`: Install and initialize Fungi, connect trusted devices, manage local and remote services, apply recipes, create `.fungi.md` service files, and diagnose results with inspect and bounded logs.
