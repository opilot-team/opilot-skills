# Opilot Skills: Repository Specification

## 1. Overview & Architecture

`opilot-skills` (`https://github.com/opilot-team/opilot-skills`) is the official public distribution repository for Opilot agent skills.

### 1.1 Architectural Boundary
- **Source of Truth**: The private development repository `opilot-team/opilot-cli` is the sole authoritative source for CLI code and skills development.
- **One-Way Publication**: Public skills are exported from an approved immutable commit of the private repository via an automated, verified publication workflow.
- **Clean Git Lineage**: This repository has its own independent Git history on branch `main`. It is not a fork or mirror of the private repository and contains no private commits, branches, issues, or development infrastructure.
- **Access Model**: Public users do not require GitHub authentication to discover or install skills. Data authorization and private package access continue to be governed by Opilot's authentication mechanisms.

---

## 2. Published Skills Catalog

The repository exposes exactly four skills organized under `skills/`:

```
skills/
├── opilot-cli-install/
│   ├── SKILL.md
│   └── references/
│       └── troubleshooting.md
├── opilot-login/
│   └── SKILL.md
├── opilot-asset-library/
│   ├── SKILL.md
│   └── references/
│       ├── filters.md
│       └── troubleshooting.md
└── opilot-radar/
    ├── SKILL.md
    └── references/
        ├── resources.md
        └── troubleshooting.md
```

### 2.1 Skill Specifications

| Skill Name | Role | Dependencies | Commands Executed |
|---|---|---|---|
| `opilot-cli-install` | Diagnostic & Setup | None | Checks Node.js version, configures npm token for GitHub Packages, installs `@opilot-team/opilot-cli`. |
| `opilot-login` | Authentication Flow | `opilot-cli-install` | Initiates device login via `opilot login` when commands return `auth_required`. |
| `opilot-asset-library` | Domain Data (Assets) | `opilot-cli-install`, `opilot-login` | Queries read-only videos, scenes, and audio via `opilot bxv assets`. |
| `opilot-radar` | Domain Data (Radar) | `opilot-cli-install`, `opilot-login` | Queries read-only pattern, event, and entity data via `opilot bxv radar`. |

### 2.2 Dependency & Handoff Invariant
Data skills (`opilot-asset-library` and `opilot-radar`) depend on the CLI being installed and authenticated. When a data skill detects that `opilot` is missing or unauthenticated, it guides the user or agent to install the companion skills (`opilot-cli-install`, `opilot-login`) rather than fabricating internal instructions or requesting credentials in chat.

---

## 3. Installation & Distribution Contract

Skills are distributed via the official [`skills`](https://www.npmjs.com/package/skills) CLI.

### 3.1 Anonymous Discovery
Any user or agent can discover available skills without GitHub credentials:
```bash
npx skills add opilot-team/opilot-skills --list
```
*Expected discovery*: Exactly the four skills listed in Section 2.

### 3.2 Installation Scenarios

#### Scenario A: Recommended All-in-One Installation (Project Scope)
```bash
npx skills add opilot-team/opilot-skills --skill '*' -a claude-code -y
```

#### Scenario B: Recommended All-in-One Installation (Global Scope)
```bash
npx skills add opilot-team/opilot-skills --skill '*' -a claude-code -g -y
```

#### Scenario C: Selective Installation (Radar)
```bash
npx skills add opilot-team/opilot-skills --skill opilot-radar --skill opilot-cli-install --skill opilot-login -a claude-code -y
```

#### Scenario D: Selective Installation (Asset Library)
```bash
npx skills add opilot-team/opilot-skills --skill opilot-asset-library --skill opilot-cli-install --skill opilot-login -a claude-code -y
```

#### Scenario E: Windows PowerShell Execution
When script execution policies restrict `npx`, invoke `npx.cmd`:
```powershell
npx.cmd skills add opilot-team/opilot-skills --skill '*' -a claude-code -y
```

### 3.3 Flag Invariants
- `--skill '*'` selects all skills in the Opilot collection.
- `-a claude-code` explicitly targets the verified Claude Code agent.
- `--all` **MUST NOT** be used (it triggers installation into all agent platforms supported by the installer rather than installing all skills for one agent).

---

## 4. Updates & Maintenance

Skills installed in local environments do not update automatically. Updates must be explicitly checked and applied:

```bash
# Check for updates across installed skills
npx skills check

# Update installed skills for current project
npx skills update -p -y

# Update installed skills globally
npx skills update -g -y
```

---

## 5. Technical Requirements & Compatibility Matrix

| Component | Tested Version | Minimum Runtime | Notes |
|---|---|---|---|
| `skills` CLI | `1.7.6` | Node.js >= 22.20 | Used for `npx skills add` and `update`. |
| `@opilot-team/opilot-cli` | Release identified in `release.json` | Node.js >= 18.0.0 | CLI runtime baseline. |
| Target Agent | Claude Code | Latest supported | Additional agents verified through runtime testing. |

---

## 6. Release Metadata & Integrity

Every approved release includes `release.json` in the root directory:
```json
{
  "releaseId": "2026.11.06-1",
  "testedCliVersion": "0.4.0",
  "publishedAt": "2026-11-06T07:45:00.000Z",
  "digest": "<sha256-payload-digest>",
  "manifest": [
    { "path": "LICENSE", "sha256": "..." },
    { "path": "README.md", "sha256": "..." },
    { "path": "skills/opilot-asset-library/SKILL.md", "sha256": "..." }
  ]
}
```

Direct modifications made to managed files on `main` are detected as drift during subsequent publication runs and must be reconciled into the private source before the next release.
