# Troubleshooting

Report only sanitized error codes, HTTP statuses and relevant hostnames. Never dump npm configuration, `~/.opilot`, environment variables, raw HTTP bodies, debug logs, tokens or authorization headers.

Installation, PAT and network settings are in the `opilot-cli-install` skill. Sign-in is in the `opilot-login` skill.

## CLI

| Failure | Action |
|---|---|
| `opilot` not found | Follow `opilot-cli-install`. Do not reinstall or ask for a PAT until it says to. |
| CLI lacks `bxv assets` | Explain that the installed version is too old and follow `opilot-cli-install`. Do not fall back to direct Bearer calls or the old skill. |

## Data command errors

| JSON error on stderr | Action |
|---|---|
| `auth_required` | Follow `opilot-login`, then retry the original command once. A second failure needs operator investigation. |
| `auth_rejected` | Do not start login. The session refreshed successfully but the server still rejected it for this command, so signing in again cannot help. Report what could not be fetched and ask the Opilot contact to check that command's server access. |
| `forbidden` | The account does not have BXV access. Ask the Opilot administrator for BXV access; do not change roles, request admin credentials or ask for a new PAT. |
| `network_error` | Check execution-host connectivity and follow the network guidance in `opilot-cli-install`. A working desktop browser does not prove a cloud sandbox can connect. Do not automatically classify every network error as a permission denial. |
| `http_error` | Report the included HTTP `status`. Server failures need operator investigation; do not claim the request succeeded. Do not change server configuration. |
| `invalid_input` | Correct arguments using filters.md. Unknown/unsupported filters require explaining the limitation and asking whether to fetch without them; never silently discard constraints. |
| `invalid_response` | The response or local credentials cannot be used. Report the safe error and ask the Opilot contact to investigate CLI/server compatibility or local configuration. Do not print raw data or credentials to diagnose it. |

If every scenes or audio command returns `auth_rejected`, the Opilot contact must check server access for those commands; do not change sign-in settings, request extra permissions or extract tokens.

For filter discovery, a 404 or malformed response is a server compatibility error, not an empty catalog. Ask the Opilot contact to check it. Do not ask the user for an internal ID; use one only if they offer it. Never substitute options sampled from data rows, invent IDs or drop a requested filter. If a later options page fails, report the subset fetched and do not claim uniqueness. Multiple exact labels require clarification; no exact label/value means not found. Re-fetch stale options. Video SKU values with surrounding whitespace are intentionally excluded; do not normalize such a value or change server/UI behavior. Inactive classifications are read-only lookup choices. Text/date keys are input guidance, so query/page on them is invalid.

The New view requires its own backend read endpoint to be deployed. Until it deploys, `opilot bxv assets new` and `filter-options new` fail with `http_error` (for example status 404) or `auth_rejected`, not an empty list. Report that the New view is unavailable and ask the Opilot contact to check the deployment; do not rebuild it from `scenes` or `audio`. `invalid_input` from `new` usually means a scene-only filter was combined with `--kind audio`, `status` was not `PENDING` or `APPROVED`, `--view` was not `live` or `trash`, `view` was passed as a `--filter` key, or a key, `--kind`, `--view` or `--page` was repeated: explain which and ask whether to fetch without the filter. A `total` at the 500-per-source cap (1,000 for `--kind all`) is a capped count, not a failure and not the library size.

Trash views (`--view trash` on `new`, `scenes`, `audio`) need the matching backend to be deployed. If it is not, the request can return `http_error`, `auth_rejected` or live rows instead of trash; do not describe that as an empty trash, and ask the Opilot contact to check the deployment. A restore request is not a CLI action: direct the user to the Asset Library page and a user with processing rights.

## Links

Signed media links from the CLI expire in about 10 minutes: the server signs Bearer responses for 600 seconds, and processing and transit use part of that. A link that no longer works (for example an expired-signature or 403 response from storage) is not a permission failure. Fetch the data again and use the new link; do not reuse or retry the old one. Do not promise an exact remaining time, and do not print links unless requested. Links in the Asset Library page in a browser use a longer lifetime and are not affected.
