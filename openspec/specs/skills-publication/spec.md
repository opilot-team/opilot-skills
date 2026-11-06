## Purpose

Provide an auditable one-way publication boundary that releases only reviewed Opilot skill files from private development source into a public repository, with repeatable validation and safe failure behavior.

## Requirements

### Requirement: Explicit immutable publication selection

Publication SHALL require an authorized manual invocation selecting an immutable resolved source commit, a public release identifier, and a tested CLI version. The source SHALL belong to the trusted private development history. The destination SHALL be separately configured and verified, not freely supplied by an untrusted caller. A source branch moving after preparation SHALL NOT change the prepared publication.

#### Scenario: Source ref moves after preparation
- **WHEN** a branch or tag resolves to commit A during preparation and later moves to commit B
- **THEN** the candidate remains the content from A and publishing B requires new preparation and approval

#### Scenario: Untrusted source or destination
- **WHEN** a request points to untrusted source history or an unauthorized destination
- **THEN** the request fails without any public write

### Requirement: Exact bounded export

The outgoing payload SHALL contain only explicitly approved files and sanitized public release metadata. Skill file contents SHALL match the selected source snapshot. Private Git history, source code, private release logs, private identifiers and URLs, environment files, credentials, and unapproved files SHALL NOT be exported. Unsafe paths, case-colliding paths, symlinks, non-regular files, and unapproved additions inside selected skill directories SHALL fail preparation rather than being silently copied.

#### Scenario: Private sibling files exist
- **WHEN** source contains `.env`, CLI code, internal documentation, or private Git metadata outside the export allowlist
- **THEN** those files are absent from the prepared payload and public commit history

#### Scenario: Unexpected file within a selected skill
- **WHEN** a new file is added under a selected skill but has not been approved in the export manifest
- **THEN** preparation stops and identifies the unapproved path without publishing its contents

#### Scenario: Symlink or escaping reference
- **WHEN** a selected path is a symlink or a required local reference resolves outside its skill package
- **THEN** validation fails and no public write occurs

### Requirement: Complete validation before disclosure

Preparation SHALL validate required skill metadata, unique names, expected collection membership, reference completeness, and discovery with the tested installer. It SHALL scan the exact outgoing content for known secret patterns and require human disclosure review. Invalid files, scan findings, scanner errors, or missing required checks SHALL prevent publication. Reports SHALL avoid printing credential values or scan matches containing secrets.

#### Scenario: Missing reference file
- **WHEN** a selected skill links to a required local reference absent from its staged package
- **THEN** validation reports the missing path and publication is blocked

#### Scenario: Metadata or discovery failure
- **WHEN** a skill lacks valid metadata, duplicates a name, or the installer fails to discover the expected four skills
- **THEN** the payload cannot advance to publication

#### Scenario: Secret scan cannot pass
- **WHEN** the scanner reports a suspected credential or cannot complete
- **THEN** publication fails closed and reports sanitized failure information without exposing the matched value publicly

### Requirement: Private approval bound to outgoing bytes

Before creating any public branch, commit, tag, or pull request, an authorized reviewer SHALL approve a private preview of the exact outgoing payload and deletions. Approval SHALL be bound to its digest, source commit, trusted tooling revision, and destination base. Payload changes or unavailable review artifacts SHALL invalidate the candidate; moving the source ref SHALL NOT change the already resolved immutable snapshot. A changed destination base SHALL block further writes unless the operation is recognized as an already completed identical release and returns read-only success. The initiating publisher SHALL NOT self-approve the required disclosure gate, and administrator bypass SHALL NOT replace approval. An unavailable approval mechanism SHALL disable publishing, not silently weaken the gate.

#### Scenario: Candidate awaits review
- **WHEN** preparation succeeds but private approval has not been granted
- **THEN** review files remain private and no candidate content is written to the public repository

#### Scenario: Payload changes after review
- **WHEN** the publication payload no longer matches the approved digest
- **THEN** publication stops and requires a fresh preparation and review

#### Scenario: Destination advances while awaiting approval
- **WHEN** the destination branch no longer matches the approved base and the candidate has not already been identically published
- **THEN** the publisher refuses to overwrite or rebase the candidate silently and requests a new preview

#### Scenario: Required reviewer gate unavailable
- **WHEN** repository configuration or subscription does not provide the required private approval gate
- **THEN** publishing remains disabled until an equivalent approved mechanism is configured

#### Scenario: Review artifact expired
- **WHEN** an approved preview artifact is missing or expired before publication
- **THEN** the publisher refuses to reconstruct and publish unseen bytes and requires a fresh preparation and approval

### Requirement: Least-privilege publication identity

Preparation SHALL NOT receive destination-write credentials. The write stage SHALL use a credential limited to the authorized destination and required publication operations, SHALL NOT execute untrusted source code, and SHALL NOT expose credentials in payloads or logs. No public workflow SHALL need credentials to read the private CLI repository. Publication SHALL NOT be triggered automatically by untrusted pull requests.

#### Scenario: Publisher credential missing or expired
- **WHEN** the authorized write stage cannot authenticate
- **THEN** it reports a sanitized failure, leaves the previous release usable, and does not request broader repository or organization permissions automatically

#### Scenario: Untrusted pull request
- **WHEN** an external contributor proposes changes to skills or publication tooling
- **THEN** that proposal cannot access the publication credential or trigger a public release before trusted review and the private disclosure gate

### Requirement: Managed synchronization without silent overwrites

The publisher SHALL modify or delete only files belonging to the validated managed inventory and newly approved non-conflicting paths. It SHALL preserve unrelated destination files. It SHALL detect direct edits to previously published managed files and reject unsafe or malformed previous inventories. It SHALL NOT silently adopt an existing nonempty unmanaged repository.

#### Scenario: Approved reference removed at source
- **WHEN** a previously managed reference is intentionally removed and its destination copy still matches the previous release
- **THEN** the next approved publication removes that obsolete copy while preserving unrelated destination files

#### Scenario: Destination managed content edited manually
- **WHEN** a public managed file differs from the recorded published hash
- **THEN** publication stops with a drift report so the accepted change can be reconciled into private source

#### Scenario: New managed path collides with an unrelated file
- **WHEN** a candidate would overwrite a destination path absent from the previous managed inventory
- **THEN** publication fails rather than overwriting that file

#### Scenario: Nonempty destination has no release inventory
- **WHEN** first publication encounters an existing nonempty target without a valid ownership inventory
- **THEN** it requires explicit bootstrap reconciliation before publishing

#### Scenario: First approved publication to the existing empty repository
- **WHEN** the approved destination base is empty and `opilot-team/opilot-skills` still contains no history
- **THEN** the first approved commit and release tag establish independent public history on `main`, without importing private parents or first publishing placeholder content

#### Scenario: Managed inventory modified directly
- **WHEN** the destination inventory is changed, including an attempt to hide a managed file edit by updating its recorded hash
- **THEN** validation against the recorded immutable release fails and publication stops rather than trusting the modified inventory

### Requirement: Idempotent releases and recoverable failures

An identical approved public content set and compatibility declaration SHALL produce no new release commit or tag even when the requested fresh release identifier differs; the existing release metadata SHALL remain unchanged. Changed content SHALL use a new release identifier. Existing release identifiers SHALL NOT be rewritten to different content, and conflicting reuse SHALL fail before any no-op shortcut. Publication SHALL update the release commit and tag atomically or fail without advancing either. It SHALL recognize an identical release that succeeded before a lost connection. Competing runs SHALL NOT overwrite one another; superseded or cancelled pending runs SHALL NOT be reported as published.

#### Scenario: Identical snapshot requested again
- **WHEN** the staged public files and compatibility declaration match the current release
- **THEN** the publisher reports a no-op and does not create an empty commit or duplicate release

#### Scenario: Release identifier reused with different content
- **WHEN** a release identifier already exists but the candidate differs
- **THEN** publication fails without force-moving the existing release

#### Scenario: Connection lost after remote success
- **WHEN** a retry verifies that the intended release tag and exact approved payload already exist on the destination main history, including when a later release has since been published
- **THEN** it reports that release as already published without writing, duplicating it, or restoring its older content over the later release

#### Scenario: Competing publication
- **WHEN** two publication runs target the same destination
- **THEN** they are serialized and any candidate based on an outdated destination requires new preparation instead of overwriting a newer release

#### Scenario: Atomic publication unavailable
- **WHEN** the destination rejects atomic branch-and-tag publication
- **THEN** the publisher stops without falling back to separate writes or reporting a partial release as successful

### Requirement: Explicit rollback and disclosure response

Rollback SHALL publish an approved known-good payload as a new release without rewriting public history. It SHALL preserve compatible CLI metadata and instruct users to update or reinstall. A disclosed credential SHALL require revocation or replacement; removing its file SHALL NOT be presented as undoing disclosure. Release records SHALL retain the immutable source and approval evidence privately.

#### Scenario: Faulty skills release
- **WHEN** maintainers approve a rollback to known-good instructions
- **THEN** a new release restores that content, preserves history, and states that already installed copies require explicit user action

#### Scenario: Credential inadvertently published
- **WHEN** a published payload is found to contain a credential
- **THEN** the response requires revoking or replacing it as well as correcting the public payload, without claiming a revert removed all copies
