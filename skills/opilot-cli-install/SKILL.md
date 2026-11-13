---
name: opilot-cli-install
description: Use when the Opilot CLI is missing, outdated or not found, or when another Opilot skill asks to prepare the CLI. Checks Node.js, installs @opilot-team/opilot-cli from GitHub Packages and guides the user through a GitHub token without it entering chat.
---

# Install the Opilot CLI

Prepare the CLI, then return to the request or skill that sent you here. No repository checkout or server configuration is needed.

## 1. Check where you are running

If the home folder is `/root` or `/home/claude`, or this skill is under `/mnt/skills`, you are in a cloud sandbox: domains may be blocked, and files do not carry over to a new chat, so the CLI and sign-in would have to be repeated in each chat. A command the user runs on their computer does not reach the sandbox, so do not ask for a PAT here. Tell the user this and recommend the Code tab in Claude Desktop, which runs on their computer. There the CLI and PAT setup is needed once, but sign-in can expire and need approving again, so do not say one sign-in is enough. Continue here only if they still want to and npm already has working authentication.

## 2. Check and install

1. Run `node --version` and `npm --version`. Require Node.js 18 or later. If missing or older, explain that Node.js must be installed from https://nodejs.org/ by the user or environment administrator, then stop before CLI installation. Do not bypass managed-environment restrictions.
2. Run `opilot --version`. If `opilot` is not found, do not assume it is missing: run `npm prefix -g` and look for it in that folder on Windows or its `bin` folder elsewhere. If it is there, add that folder to `PATH` only for the commands you run, and do this for every later `opilot` command. Do not change system or user `PATH` settings. Check the published version using `npm view @opilot-team/opilot-cli version --registry=https://npm.pkg.github.com`. Do not dump npm configuration, environment variables or debug logs: these can contain secrets.
3. If installed and published versions match, skip installation and do not ask for a PAT. If the published version cannot be checked but the installed CLI runs, continue with it and tell the user it may not be the latest. Do not request a PAT merely to check the version in that case.
4. If missing or older, use existing npm authentication when available. Configure only the package scope with `npm config set @opilot-team:registry https://npm.pkg.github.com --location=user`; leave the default npm registry unchanged. Install `npm install -g @opilot-team/opilot-cli@latest`. Verify `opilot --version` afterward, using the `PATH` handling in step 2 if needed. Do not publish or change the package registry.
5. If installation needs authentication and no working PAT is configured, use the PAT steps below. For 403/404, ask for package access instead of repeatedly requesting tokens. See [troubleshooting](references/troubleshooting.md).

If the skill that sent you needs a command the installed CLI lacks, explain that a newer CLI is needed rather than inventing a replacement command.

## 3. PAT setup only when needed

The Opilot contact must grant the user's GitHub account read access to the private `@opilot-team/opilot-cli` package. Prefer package-level access where available; do not ask for repository or organization administration.

Give this link: https://github.com/settings/tokens/new?description=opilot-cli&scopes=read:packages

Ask the user to create a **classic** PAT with only **read:packages** and a short expiry, and to copy it. It can read packages their account can access, but it is not an Opilot login. They can revoke it at https://github.com/settings/tokens. Do not request `repo`, write or delete scopes.

**Never ask for the PAT in chat.** The user sets it themselves. Send these steps in their language, with the command in its own code block:

1. Open a terminal on your computer. On Windows, open PowerShell from the Start menu.
2. Copy this command and replace `PASTE_TOKEN_HERE` with your token. Keep the quotes.

   On Windows, give only this form. PowerShell blocks plain `npm` under its default script policy:

   ```text
   npm.cmd config set "//npm.pkg.github.com/:_authToken" "PASTE_TOKEN_HERE" --location=user
   ```

   On macOS or Linux, give only this form:

   ```text
   npm config set "//npm.pkg.github.com/:_authToken" "PASTE_TOKEN_HERE" --location=user
   ```

3. Press Enter. If nothing is printed, that is correct. Then tell me "done". Do not send me the token.

Do not run this command yourself, and do not read the user's terminal. If the user reports a script policy error, give the `npm.cmd` form; do not ask them to change the execution policy. After they reply, run the installation above and report success. Never read the npm config back into output. A 401 means the token is invalid, expired or the placeholder was not replaced: explain revocation and repeat these steps with a new PAT, not broader scopes.

If the user pastes a PAT in chat anyway, do not use it. Tell them chat history may retain it, ask them to revoke it and create a new one, then repeat these steps.

## 4. Network blocks

Use [troubleshooting](references/troubleshooting.md). For proxy/egress denial, recommend the Code tab first. To use chat anyway, guide the user to **Settings → Capabilities → Create and edit files → Allowed websites → Manage** (an administrator's Organization settings on Team or Enterprise). Labels can vary by version. Add bare domains separately: `npm.pkg.github.com`, `opilot.me`, `auth.opilot.me`. Retry only after the user confirms the change. If a package download identifies another blocked host, ask the administrator to approve that exact host; do not guess wildcards. Never bypass organization policy, disable TLS verification or expose proxy credentials.
