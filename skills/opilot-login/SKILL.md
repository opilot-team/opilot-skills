---
name: opilot-login
description: Use when an opilot command returns auth_required, or when the user asks to sign in to Opilot. Runs device login, shows the approval link and code in chat, and returns to the original request. Do not use for auth_rejected.
---

# Sign in to Opilot

Sign in only when an `opilot` command returned `auth_required` or the user asked. Then return to the request or skill that sent you here. If `opilot` cannot be found, use the `opilot-cli-install` skill first. If that skill is not available, tell the user its name. Do not guess its steps, and do not ask for a token in chat. Then choose by where you are running:

- **Local coding agent** (such as Claude Code): the public source `opilot-team/opilot-skills` has it. Confirm the agent and scope with the user, then give them (or, with their approval, run) `npx skills add opilot-team/opilot-skills --skill opilot-cli-install -a <agent> -y`, adding `-g` only for a global install. Never use `--all`, which installs into every agent. The skills installer needs Node.js 22.20 or later even though the CLI itself needs 18, and on Windows PowerShell use `npx.cmd` in place of `npx` if script policy blocks it. Installing a skill does not install the CLI or grant package or data access. If the new skill does not load, ask the user to start a new session.
- **Upload-only surface** (accepts uploaded skill zips but does not load local skill folders): ask them to upload the `opilot-cli-install` zip. Running `npx skills` on their computer does not install into that surface.

First sign-in, sign-in after an expired session and a new code after an expired code all follow these same steps:

1. Start `opilot login --device` as a background shell command whose output you can read. Do not use a Terminal panel or terminal tab tool when a background command is available.
2. Read its output, then immediately send this chat message with the full approval URL and code copied exactly, in the user's language:

   ```text
   Open this link, sign in and approve this code. I will fetch the data after approval.
   Link: <full https URL from the output>
   Code: <code from the output>
   ```

   Tool output may be collapsed or hidden from the user, so the message itself must contain the `https://` link. Never say the link is attached, above or already provided, and never send them to a terminal tab to find it. If you cannot read the URL, say so and stop the process instead of waiting silently. The user signs in in their own browser. Never fabricate a link or ask them to paste a token.
3. Keep the same process running until it finishes; do not restart it while waiting. If persistent execution is unavailable, explain the environment limitation and stop.
4. Successful login may include a telemetry setup warning. Treat login as successful; do not ask the user to install or repair telemetry or run telemetry commands. Login's existing telemetry behavior is unchanged. Raise unexpected telemetry provisioning with the Opilot contact rather than changing it.
5. Retry the original data command **once** after successful login. A repeated `auth_required` needs operator investigation, not a login loop. Do not retry data after failed or denied login.

If login fails, is denied or times out, stop that process and offer a new attempt rather than launching overlapping logins. A successful login is not proof of successful data retrieval: only show data a successful data command returned.

## Credentials

Let the CLI manage `~/.opilot` credentials. Never open, print, copy or extract them. Do not call APIs with a token yourself. Do not ask for issuer, client ID, password, access token or admin credentials. Report only sanitized error codes, and do not print raw login output beyond the link and code.

The Code tab keeps sign-in on the user's computer; a cloud sandbox does not keep it for a new chat. Sign-in can expire, so do not promise it lasts, and do not modify Keycloak or request `offline_access`.
