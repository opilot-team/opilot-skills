# Troubleshooting installation

Report only sanitized error codes, HTTP statuses and relevant hostnames. Never dump npm configuration, `~/.opilot`, environment variables, raw HTTP bodies, debug logs, tokens or authorization headers.

## Installation and package access

| Failure | Action |
|---|---|
| Node.js missing or below 18 | Explain how to install Node.js from https://nodejs.org/ or contact the managed environment administrator. Stop before CLI installation. |
| Latest-version lookup fails but installed CLI runs | Continue with installed CLI; state it may not be latest. Do not ask for a PAT just to perform that lookup. |
| npm 401 | PAT is missing, invalid or expired. If needed, ask for a new classic PAT with only `read:packages` and short expiry. Revoke the old one at https://github.com/settings/tokens. Never ask for broader scopes. |
| npm 403 or 404 | Ask the Opilot contact to grant the GitHub account package read access. A valid PAT alone does not grant package access. The contact may also need to approve organization SSO for the PAT. Do not retry with admin credentials or publish the package publicly. |
| Permission denied during global install | Explain the environment's installation-permission issue. Ask its administrator to provide an approved writable npm prefix; do not silently elevate privileges. |
| User pastes a PAT in chat | Do not use it. Ask them to revoke it, create a new one and run the `npm config set` command in SKILL.md themselves. |
| `opilot` not found after installation | Run `npm prefix -g`. If the CLI is in that folder (or its `bin` folder outside Windows), add it to `PATH` only for the commands you run. Do not reinstall, request a PAT or change system settings. |

The package scope uses `https://npm.pkg.github.com`; public dependency resolution retains the existing default registry. PAT storage is limited to the user's npm configuration. Do not write credentials to the project, the skill, exports or temporary scripts. Never include token values in examples or reports. The user sets the PAT in their own terminal, so it stays out of chat; their terminal history may retain the command, so keep the expiry short and explain revocation.

## Where Claude runs

Chat runs commands in a cloud sandbox, and the Code tab runs them on the user's computer. Upgrading the plan does not move chat onto the computer.

| Tested 2026-11-03 | Network to Opilot | Files in a new chat |
|---|---|---|
| Code tab, paid plan | Works | Kept |
| Chat, Pro, default websites | Blocked (proxy 403) | Not tested |
| Chat, Pro, domains added | Works | **Lost** |
| Chat, Free, default websites | Blocked (`ENOTFOUND`) | Not tested |

Recommend the Code tab. In chat, the CLI and sign-in would have to be repeated in each new chat, and the PAT command cannot reach the sandbox. Do not tell the user that upgrading fixes this, and do not claim what a plan allows beyond this table.

## Network settings for chat

For `ENOTFOUND`, proxy CONNECT denial, egress refusal or `connect_rejected`, the sandbox is blocking the domain. If the user still wants chat, guide them to **Settings → Capabilities → Create and edit files → Allowed websites → Manage**. On Team or Enterprise this may be under **Organization settings → Capabilities**, and only an owner can change it. Labels can differ by app version. If the option is missing, use the Code tab. Add these bare domains separately, and do not choose all websites:

- `npm.pkg.github.com`
- `opilot.me`
- `auth.opilot.me`

If an actual npm dependency or tarball request names another blocked host, ask the workspace administrator to authorize that exact host according to policy. Do not guess broad wildcards or reroute around the proxy. Retry after the user confirms the change. If settings are locked, the workspace administrator must approve access.

A proxy CONNECT 403 occurs before the Opilot API is reached and is **not** a BXV role denial. Do not solve it with elevated credentials. For certificate errors, use the host's approved trust configuration; never disable TLS verification or print proxy environment variables.

## New chats

In a new chat, recheck the CLI before asking for a PAT. See "Where Claude runs" for what is kept; do not claim more than that table shows.
