# Project Context: opilot-skills

## Purpose

`opilot-skills` is the public distribution repository for Opilot AI agent skills (`opilot-team/opilot-skills`).
It enables developers and local coding agents to discover, install, and update Opilot skills without requiring GitHub authentication or repository access to the private CLI codebase.

Key responsibilities:
- Host approved public release snapshots of Opilot skills.
- Serve as the public registry for `npx skills add opilot-team/opilot-skills`.
- Maintain complete, self-contained reference files for each published skill.
- Provide clear user and agent documentation regarding prerequisites, installation scopes (project vs. global), dependencies, and update workflows.

## Source & Publication Architecture

- **Private Source Repository**: `opilot-team/opilot-cli` (authoritative source for code, tests, and skills).
- **Public Distribution Repository**: `opilot-team/opilot-skills` (this repository).
- **One-Way Publication Boundary**: Skills are authored, tested, and reviewed in the private CLI repository. Approved files are exported here via an audited, digest-locked publication process.
- **Git History Boundary**: This repository maintains its own clean Git history and contains no private commit history, CLI source code, internal credentials, or development infrastructure.
- **Contribution Model**: Community feedback and corrections are incorporated into the private authoritative source repository prior to subsequent releases.

## Included Skills

| Skill | Directory | Purpose | Required Dependencies |
|---|---|---|---|
| `opilot-cli-install` | `skills/opilot-cli-install` | Guides installation and setup of the `@opilot-team/opilot-cli` package from GitHub Packages. | None |
| `opilot-login` | `skills/opilot-login` | Guides device sign-in flow when CLI commands return `auth_required`. | `opilot-cli-install` |
| `opilot-asset-library` | `skills/opilot-asset-library` | Queries read-only Asset Library videos, scenes, and audio via `opilot bxv assets`. | `opilot-cli-install`, `opilot-login` |
| `opilot-radar` | `skills/opilot-radar` | Queries read-only BXV Radar pattern, event, and entity data via `opilot bxv radar`. | `opilot-cli-install`, `opilot-login` |

## Tech Stack & Standards

- **Skill Specification**: Standard Agent Skills format (`SKILL.md` with YAML frontmatter `name` and `description`, markdown instructions, and local `references/*.md`).
- **Installer**: `skills` CLI via `npx` (tested with `skills@1.7.6`, requires Node.js >= 22.20).
- **Target CLI**: `@opilot-team/opilot-cli` (requires Node.js >= 18).
- **Verified Agent**: Claude Code (`-a claude-code`).
- **Release Metadata**: `release.json` (contains release ID, tested CLI version, release timestamp, manifest file list, and content digest).

## Project Conventions

- File structure:
  - `skills/<skill-name>/SKILL.md`: Skill entrypoint.
  - `skills/<skill-name>/references/*.md`: Detailed reference documents linked relatively from `SKILL.md`.
  - `README.md`: Public-facing onboarding, installation, and update guide.
  - `LICENSE`: Open source license.
  - `release.json`: Release manifest and integrity metadata.
- All files are UTF-8 text with LF line endings.
- Relative links within skills must be contained strictly within the respective skill directory.
