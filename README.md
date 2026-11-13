# Opilot skills

Agent skills for working with Opilot data from a local coding agent. This repository is the public distribution of the skills collection at `opilot-team/opilot-skills`; the skills are developed in a separate private source repository and published here as reviewed snapshots.

## Skills

| Skill | Purpose | Needs |
|---|---|---|
| `opilot-cli-install` | Prepares the Opilot CLI: Node.js check, install from GitHub Packages, package token setup | - |
| `opilot-login` | Device sign-in to Opilot after a command reports `auth_required` | `opilot-cli-install` |
| `opilot-asset-library` | Read-only Asset Library videos, scenes and audio through `opilot bxv assets` | `opilot-cli-install`, `opilot-login` |
| `opilot-radar` | Read-only BXV Radar data through `opilot bxv radar` | `opilot-cli-install`, `opilot-login` |
| `opilot-ads` | Read-only Ads Ranking, Ads Planning and Video Metrics through `opilot bxv ads-ranking`, `ads-planning` and `video-metrics` | `opilot-cli-install`, `opilot-login` |

The three data skills hand off to `opilot-cli-install` and `opilot-login`, so install those support skills together with them.

## Requirements

- **Skills installer:** the commands below use the [`skills`](https://www.npmjs.com/package/skills) installer through `npx`. It was tested at version 1.7.6, which requires **Node.js 22.20 or later**. To reproduce the tested behavior exactly, write `npx skills@1.7.6` instead of `npx skills`.
- **Opilot CLI:** the skills drive the `opilot` CLI, which has its own, lower requirement of Node.js 18 or later. Having Node.js 18 is enough for the CLI but not for the skills installer.
- **Agent:** the commands below name the Claude Code agent (`-a claude-code`). Replace it only with another agent name that the installer supports; this README does not cover other agents.

## Claude Desktop plugin marketplace

The same skill folders are also bundled as the root plugin **`opilot-bxv`** in marketplace **`opilot-team`**. In Claude Desktop, open **Customize → Plugins → Add → Add marketplace**, enter `https://github.com/opilot-team/opilot-skills`, then install `opilot-bxv`. Availability and labels depend on account policy and app version.

For Claude Code, the corresponding marketplace commands are:

```text
/plugin marketplace add opilot-team/opilot-skills
/plugin install opilot-bxv@opilot-team
```

Do not keep standalone uploads and the bundled copies enabled together during testing. In a fresh session, select a plugin-qualified skill such as `/opilot-bxv:opilot-radar` and verify its full instructions and references load.

Plugin releases use the publication release ID, independently of the CLI version. Skill edits, approved additions and removals arrive as a complete plugin update after marketplace syncing. Use **Check for updates**; personally added github.com marketplaces may offer **Sync automatically**. Updates are not necessarily immediate; verify in a new session. Standalone skill uploads are not removed by updating the plugin. Removing instructions does not revoke backend permissions.

Installing the plugin does not install/update the Opilot CLI or grant package or BXV access. Desktop Chat may execute in a cloud sandbox, unlike a local Code session; follow `opilot-cli-install` rather than assuming local credentials are available.

Local manifest validation is not proof of live marketplace installation, update delivery or production data access. Organization-managed syncing has separate admin/repository requirements and is not configured by these commands.

## Install

Check that the collection can be found. No GitHub login is needed:

```bash
npx skills add opilot-team/opilot-skills --list
```

You should see exactly these five skills: `opilot-ads`, `opilot-asset-library`, `opilot-cli-install`, `opilot-login` and `opilot-radar`.

### All five skills (recommended)

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

### Only Radar, Asset Library or Ads

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

Ads, project scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-ads --skill opilot-cli-install --skill opilot-login -a claude-code -y
```

Ads, global scope:

```bash
npx skills add opilot-team/opilot-skills --skill opilot-ads --skill opilot-cli-install --skill opilot-login -a claude-code -g -y
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

## Public skills are not access

Installing these skills only gives your agent instructions. It does not give you the CLI or any Opilot data:

1. **Public skills:** anyone can install them from this repository without credentials.
2. **CLI package:** `@opilot-team/opilot-cli` is a private package on GitHub Packages. Your GitHub account needs read access to it, which the Opilot contact grants, and a read-only package token. The `opilot-cli-install` skill guides this; never paste a token into chat.
3. **Opilot data:** signing in with `opilot login --device` uses your Opilot account. What you can read depends on your Opilot roles, and a skill cannot widen it.

If you can install the skills but not the package, or the data command is denied, ask your Opilot contact for access.

Uploading skills as ZIP files into a chat client that does not load local skill folders is a separate route, handled by your Opilot contact. Running `npx skills` on your computer does not install into such a client.

## Releases

Each release lists its tested CLI version and the files it contains in `release.json`. A skill that uses a new CLI command is released only after that CLI version is available to its users. This README does not promise that every agent behaves the same way with the skills.

## Source and changes

The skills are developed and maintained by the Opilot team in a private source repository. This repository is a one-way publication of reviewed snapshots for installation only.

- This repository does not accept external issues or pull requests. If you use Opilot and something is wrong, tell your Opilot contact.
- Install only from `opilot-team/opilot-skills`. Copies or forks published elsewhere are not maintained by Opilot and may contain altered instructions.
- Never post credentials, tokens or customer data anywhere in this repository.

## License

[Opilot Skills License](LICENSE). You may install and use these skills only to access Opilot services with an authorized Opilot account; redistribution and modified copies are not permitted.
