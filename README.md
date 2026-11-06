# Opilot skills

Agent skills for working with Opilot data from a local coding agent. This repository is the public distribution of the skills collection at `opilot-team/opilot-skills`; the skills are developed in a separate private source repository and published here as reviewed snapshots.

## Skills

| Skill | Purpose | Needs |
|---|---|---|
| `opilot-cli-install` | Prepares the Opilot CLI: Node.js check, install from GitHub Packages, package token setup | - |
| `opilot-login` | Device sign-in to Opilot after a command reports `auth_required` | `opilot-cli-install` |
| `opilot-asset-library` | Read-only Asset Library videos, scenes and audio through `opilot bxv assets` | `opilot-cli-install`, `opilot-login` |
| `opilot-radar` | Read-only BXV Radar data through `opilot bxv radar` | `opilot-cli-install`, `opilot-login` |

`opilot-asset-library` and `opilot-radar` hand off to the other two skills, so install those together with them.

## Requirements

- **Skills installer:** the commands below use the [`skills`](https://www.npmjs.com/package/skills) installer through `npx`. It was tested at version 1.7.6, which requires **Node.js 22.20 or later**. To reproduce the tested behavior exactly, write `npx skills@1.7.6` instead of `npx skills`.
- **Opilot CLI:** the skills drive the `opilot` CLI, which has its own, lower requirement of Node.js 18 or later. Having Node.js 18 is enough for the CLI but not for the skills installer.
- **Agent:** the commands below name the Claude Code agent (`-a claude-code`). Replace it only with another agent name that the installer supports; this README does not cover other agents.

## Install

Check that the collection can be found. No GitHub login is needed:

```bash
npx skills add opilot-team/opilot-skills --list
```

You should see exactly these four skills: `opilot-asset-library`, `opilot-cli-install`, `opilot-login` and `opilot-radar`.

### All four skills (recommended)

Project scope (the current project only):

```bash
npx skills add opilot-team/opilot-skills --skill '*' -a claude-code -y
```

Global scope (every project for your user account):

```bash
npx skills add opilot-team/opilot-skills --skill '*' -a claude-code -g -y
```

`--skill '*'` selects every skill in this collection. Always name the agent with `-a`. Do not use `--all`: it installs into every agent the installer knows, not only the Opilot skills for one agent.

Project scope is the default and installs into the current directory; run the command from your project folder. Global scope is chosen with `-g`.

### Only Radar or only Asset Library

Always include `opilot-cli-install` and `opilot-login` with a data skill.

Radar, project scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-radar --skill opilot-cli-install --skill opilot-login -a claude-code -y
```

Radar, global scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-radar --skill opilot-cli-install --skill opilot-login -a claude-code -g -y
```

Asset Library, project scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-asset-library --skill opilot-cli-install --skill opilot-login -a claude-code -y
```

Asset Library, global scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-asset-library --skill opilot-cli-install --skill opilot-login -a claude-code -g -y
```

### Windows PowerShell

If PowerShell blocks `npx` because of its script policy, call `npx.cmd` instead. You do not need to change the execution policy. For example:

```powershell
npx.cmd skills add opilot-team/opilot-skills --list
npx.cmd skills add opilot-team/opilot-skills --skill '*' -a claude-code -y
```

Every `npx` command in this README works the same way with `npx.cmd`.

## Update

Installed skills do not change on their own. To receive a newer release, update them yourself:

```bash
npx skills check
npx skills update
npx skills update -p
npx skills update -g
```

`check` reports available updates, `update` updates installed skills (it asks for a scope unless you pass `-p` for project or `-g` for global), and the `-y` flag skips that question. You can also name skills, for example `npx skills update opilot-radar`. To start over, run the install command again.

## License

See [LICENSE](LICENSE) for terms.
