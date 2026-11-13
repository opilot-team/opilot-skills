# Commands and filters

Lists that take paging: `ads-ranking patterns`, `ads-ranking videos`, `ads-ranking logic`, `ads-planning rows`, `video-metrics videos`. They take `--page`, `--page-size` (25, 50, 100 or 200), `--sort-by`, `--sort-dir` (`asc` or `desc`) and repeated `--filter key=value`, and they return `rows` with `total`, `page` and `pageSize`. A page past the end has empty `rows` with the real `total`.

`pattern-videos`, `logic-videos` and `ads-planning videos` return all their results in `videos`, with no `total`. They take no paging, sort or `--filter` options, and passing one returns `invalid_input`.

## `opilot bxv ads-ranking`

This is the page `opilot.me/bxv/ads-ranking`.

### `patterns`

The pattern tabs. Options: `--view`, `--from`, `--to`, `--detail`.

| `--view` | Tab |
|---|---|
| `ta` (default) | TA |
| `insight` | Insight |
| `hookVoice` | Hook Voice |
| `hookAngle` | Hook Angle |
| `hookVisual` | Hook Visual |
| `structure` | Script Structure |
| `structureFormula` | Structure × Formula |
| `message` | Message |

Without `--from` and `--to`, it uses the page's default of the last 30 days of data, and `window` reports it. The scores use 7, 14 and 28 day periods ending at `--to`. `--from` only changes the `coverage` figures.

Filters: `product` (name as in the rows), `state` (`STAR`, `CASH COW`, `QUESTION MARK`, `DOG`).

`allocation`, `strength`, `growth` and `confidence` are scores from 0 to 100 within the product, not shares of budget. `coverage` describes the whole account for the window and ignores `--filter`.

Sort keys: `allocation` (default), `strength`, `growth`, `confidence`, `recentQualifiedExecutions`, `recentWins`, `independentReplication`, `strengthSpend`, `strengthRoas`, `product`, `parentName`, `patternName`.

`--detail` adds `raw`, `rank` and `basis`: how a score was built. Use it only when the user asks why.

### `videos`

The Video tab. Options: `--from`, `--to`. Filter: `product`. It includes each video's tags and Facebook link.

Sort keys: `spend28` (default), `roas28`, `spend14`, `roas14`, `spendPrev14`, `spendLifetime`, `spendShare28`, `roasIndex28`, `roasMomentum`, `label`, `product`.

### `pattern-videos`

The videos behind one `patterns` row. Required: `--view`, `--product`, `--pattern` (the row's `patternId`, unchanged, even if it contains `|`). Optional: `--to`.

### `logic`

The "Điểm logic tổ hợp" tab: judged logic per combination, with no date period. Rows keep their overall `rank` when filtered. `score` is Jev's score. `llmScore` is a separate second opinion, only for some combinations, and the two are not combined.

Filters: `productId`, `taId`, `insightId`, `structureId`, `formulaId`, `hookVoiceId`, `hookAngleId` (codes, found with `filters logic`), and `madeMin`, `madeMax` (videos already made).

Sort keys: `score` (default), `llmScore`, `avgSpend`, `insightName`, `formulaName`, `hookAngleName`, `daily`.

### `logic-videos`

Brief videos of one combination. Required: `--ta`, `--insight`, `--formula`, `--angle`. Pass the row's `productId` as `--product`, as the page does; without it the list includes other products. Optional: `--library` (also library videos).

### `filters <target>`

Targets: `patterns`, `videos`, `logic`. Options: `--key`, `--query`, `--page`. `--from` and `--to` apply to `patterns` and `videos`, and `--view` to `patterns`, because their product lists come from those rows. Pass the same date options as the list you will fetch, including `--from` when no `--to` is given: that leaves the scoring end date at the newest applicable data instead of seeding the default window. Neither date option applies to `logic`.

## `opilot bxv ads-planning`

This is the page `opilot.me/bxv/ads-planning`. It reads the newest finished run. No command starts a run.

### `runs`

`latest` is the newest run of any status, and `latestDone` is the one `rows` reads. `stale: true` means a run was abandoned mid-way; say so instead of calling it in progress.

### `rows`

| `--tier` | Meaning |
|---|---|
| `proven` (default) | Has real qualified videos in the period |
| `near` | Estimated from similar proven combinations within the distance threshold |
| `unaffected` | Too far from any proven combination to estimate |

`--near-threshold` (above 0, at most 2) moves the line between `near` and `unaffected`. It defaults to the run's own value, and `tierCounts` shows the three counts at that value. `--detail` adds `scoreParts`.

Filters: `productId`, `taInsightId` (such as `TA02-IN04`), `formulaId`, `hookVoiceId`, `hookAngleId`, `bcgState` (`STAR`, `CASH_COW`, `QUESTION_MARK`, `DOG`). Note the underscores, which differ from `patterns`.

Sort keys: `allocation` (`proven` default), `strength` (`near` default), `distance` (`unaffected` default), `growth`, `jev`, `llm`, `productId`, `taInsightId`, `formulaId`, `hookAngleId`, `spend`, `roas`, `qualifiedVideos`, `winVideos`, `observedVideos`.

### `videos`

Brief and library videos of one combination. Required: `--ta`, `--insight`, `--formula`, `--angle`. Pass the row's `productId` as `--product`, as the page does; without it the list includes other products. A row's `taInsightId` of `TA02-IN04` means `--ta TA02 --insight IN04`.

### `filters rows`

Options: `--key`, `--query`, `--page`.

## `opilot bxv video-metrics`

This is the page `opilot.me/bxv/video-metrics`.

### `videos`

Spend and ROAS per video over 14 days, the previous 14 days, 28 days and lifetime, all ending at the newest data (`anchor`). It has no date option. For another end date, use `ads-ranking videos --to`.

The page opens on one product. This command returns every product unless you filter by `product`.

`spendScores` holds the charts' log score from 0 to 100 for each spend period, and `spendCaps` holds each product's 100 mark. Sort keys are the same as `ads-ranking videos`.

### `filters videos`

Options: `--key`, `--query`, `--page`.
