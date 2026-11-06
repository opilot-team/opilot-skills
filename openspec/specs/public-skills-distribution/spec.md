## Purpose

Allow the community to discover and install Opilot skills from a public Git source while keeping the CLI development repository authoritative and private, without changing package or data access controls.

## Requirements

### Requirement: Private source and public distribution ownership

The four skills SHALL remain editable in the private CLI repository. The public distribution repository SHALL be the organization-owned `opilot-team/opilot-skills`, with its own Git history containing only approved public files. Publication SHALL NOT change the visibility of the CLI repository, mirror its history, or require public infrastructure to read private source. Public contributions SHALL be incorporated into the private source before subsequent publication. No separate npm skills package SHALL be required.

#### Scenario: Publishing a private development snapshot
- **WHEN** an approved private snapshot is published
- **THEN** users can read the approved skills and public documentation but cannot access CLI source, private commit history, private workflow artifacts, or credentials through the distribution repository

#### Scenario: Community correction
- **WHEN** a correction is proposed against the public skill content
- **THEN** maintainers apply the accepted correction to the private authoritative source before releasing it, rather than maintaining two independently edited versions

### Requirement: Anonymous complete skill installation

The public source SHALL expose `opilot-cli-install`, `opilot-login`, `opilot-asset-library`, and `opilot-radar` through `npx skills add opilot-team/opilot-skills --list` without GitHub authentication. Installation SHALL include each selected skill's required references with relative paths preserved. The documented default SHALL install all four skills for an explicitly selected agent and explain project versus global scope; it SHALL NOT use `--all` as a synonym for installing only the Opilot collection into one agent.

#### Scenario: Discovery without credentials
- **WHEN** a user with the documented installer prerequisites but no GitHub credentials lists the public source
- **THEN** exactly the four released Opilot skill names are discoverable without granting access to the private CLI repository

#### Scenario: Complete project installation
- **WHEN** a user installs the collection for Claude Code at project scope in a clean directory
- **THEN** all four skills and their referenced files are available to that project without creating a global installation

#### Scenario: Global installation
- **WHEN** a user explicitly selects global installation for an agent
- **THEN** the collection is installed at that agent's supported user-level location rather than requiring a project dependency

### Requirement: Dependency-complete handoff guidance

Public documentation and the skills SHALL identify `opilot-cli-install` and `opilot-login` as required supporting skills for Radar and Asset Library. Selective installation examples SHALL include those dependencies. Missing-skill guidance SHALL distinguish supported local coding agents from upload-only surfaces, name the missing dependency, and never invent its instructions or request a PAT in chat. `opilot-login` SHALL also provide this guidance when it needs an unavailable install skill.

#### Scenario: Selective Radar installation
- **WHEN** a user follows the documented selective Radar command
- **THEN** Radar, CLI installation, and login skills are installed together

#### Scenario: Missing support skill in a local agent
- **WHEN** a data or login skill requires another Opilot skill that is unavailable in a supported local coding agent
- **THEN** it names the missing skill and provides the public-source installation route for the selected agent and scope, without silently installing into all agents or fabricating the missing workflow

#### Scenario: Upload-only client
- **WHEN** the client accepts uploaded skill ZIPs but does not load local agent skill directories
- **THEN** guidance retains the existing separate-ZIP upload route and does not claim that running `npx skills` on the user's computer installs into that client's cloud chat

### Requirement: Honest prerequisites and access boundaries

Documentation SHALL distinguish the tested skills installer prerequisites from the CLI runtime requirements. It SHALL state the tested installer version and required Node version, provide the Windows `npx.cmd` alternative for script-policy failures, and distinguish public skill access from private GitHub Packages access and Opilot data authorization. This change SHALL NOT alter the CLI engine requirement, package registry/visibility, npm package contents, login, or backend permissions.

#### Scenario: User has Node supported only by Opilot
- **WHEN** the user has Node 18 but the tested skills installer requires Node >=22.20
- **THEN** onboarding explains the higher installer requirement instead of claiming the CLI's Node minimum is sufficient or changing system policy automatically

#### Scenario: Skill installed but package access absent
- **WHEN** a user can install public skills but cannot read the private CLI package
- **THEN** guidance explains the existing package-access requirement without granting repository access, requesting broader credentials, or treating skill installation as CLI installation

#### Scenario: Data permission denied
- **WHEN** the user has installed skills and CLI but is denied an Opilot data operation
- **THEN** the existing authentication and authorization handling remains authoritative and public distribution does not bypass it

### Requirement: Compatible releases and explicit client updates

Every public skill release SHALL identify a tested CLI version already available to its intended users. Documentation-only releases SHALL be possible without a new CLI package version. Public documentation SHALL describe updating installed skills and SHALL NOT imply automatic propagation or universal functional agent compatibility without evidence.

#### Scenario: New command is not released
- **WHEN** a candidate skill instructs a command not available in the claimed published CLI version
- **THEN** that candidate is not approved for public release

#### Scenario: Documentation-only correction
- **WHEN** instructions are corrected without changing supported CLI commands
- **THEN** a new skills release can retain the same tested CLI version

#### Scenario: Existing installation receives an update
- **WHEN** a second approved snapshot changes a skill and the user invokes the documented skills update command
- **THEN** the installed skill and its references reflect that snapshot; users who do not update are not claimed to have received it

#### Scenario: Additional agent support
- **WHEN** documentation claims functional support for an agent beyond the initially verified Claude Code workflow
- **THEN** that claim is supported by a corresponding runtime check rather than discovery or file-copy success alone
