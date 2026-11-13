# Troubleshooting

Report only sanitized error codes, HTTP statuses and relevant hostnames. Never dump npm configuration, `~/.opilot`, environment variables, raw HTTP bodies, debug logs, tokens or authorization headers.

Installation, PAT and network settings are in the `opilot-cli-install` skill. Sign-in is in the `opilot-login` skill.

## CLI

| Failure | Action |
|---|---|
| `opilot` not found | Follow `opilot-cli-install`. Do not reinstall or ask for a PAT until it says to. |
| CLI lacks `bxv radar` | Explain that the installed version is too old and follow `opilot-cli-install`. Do not call the API directly. |

## Data command errors

| JSON error on stderr | Action |
|---|---|
| `auth_required` | Follow `opilot-login`, then retry the original command once. A second failure needs operator investigation. |
| `auth_rejected` | Do not start login. The session refreshed but the server still rejected it, so signing in again cannot help. Report what could not be fetched and ask the Opilot contact to check server access. |
| `forbidden` | The account does not have BXV access. Ask the Opilot administrator for BXV access; do not change roles, request admin credentials or ask for a new PAT. |
| `network_error` | Check execution-host connectivity and follow the network guidance in `opilot-cli-install`. A working desktop browser does not prove a cloud sandbox can connect. |
| `http_error` 404 | The server does not support this request yet, or the item does not exist. This is not an empty result. Ask the Opilot contact to check it if every command fails this way. |
| `http_error` 400 | The server rejected the request, for example a resource an older backend does not support, or a bad filter value. Report the status and correct the input if you can. Never report it as no data. |
| `http_error` 5xx | Server failure needing operator investigation. Do not claim the request succeeded. |
| `invalid_input` | Correct arguments using `references/resources.md`. Unsupported filters require explaining the limit and asking whether to fetch without them. |
| `invalid_response` | The response does not match this CLI, often because the CLI and server versions differ. Ask the user to update and the Opilot contact to check compatibility. Do not print raw data. |

A successful login is not proof of successful data retrieval. Only show data a successful command returned.

When `filters` returns `hasMore: true`, read the next pages before saying a value does not exist. If a later page fails, report what you fetched and do not claim uniqueness.
