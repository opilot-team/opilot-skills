# Resources and filters

Use only these commands and keys. The server's `opilot bxv radar filters <target>` is the source of truth; this page is a guide. Dates are `YYYY-MM-DD` in Vietnam time.

## Discover filters and options

```bash
opilot bxv radar filters creatives
opilot bxv radar filters keywords --key category
opilot bxv radar filters creatives --key patternTa --query "Mẹ" --page 2
opilot bxv radar filters patterns --key lib_ta
```

Targets: `keywords`, `creatives`, `advertisers`, `posts`, `patterns`, `publishers`, `jobs`, `saved`.

Without a key, the result is `{ target, filters }`. With a key that has options, it is `{ target, filter, options, page, pageSize, total, hasMore }`, 100 options per page. A text, date or number filter returns only `{ target, filter }`, which gives input rules and is not an empty list.

Sort fields and directions are enumerable filters too. Resolve their allowed values with `--key`, for example `opilot bxv radar filters creatives --key metaSortBy`. For posts, use the selected platform's sort keys and pass the same livestream scope when discovering sort values.

Publisher options come from `--key publisherId` on the `creatives`, `posts` or `publishers` targets. Without `--filter platform=...` they cover the backend default (Meta for `publishers` and `creatives`, TikTok for `posts`), so for another platform pass the scope: `opilot bxv radar filters publishers --key publisherId --filter platform=TIKTOK`, or find the publisher through `opilot bxv radar publishers --filter platform=TIKTOK` and use its `id`. A posts publisher filter also accepts `publisherId=__missing__` for posts without a publisher.

Publisher options respect the selected pattern IDs and livestream scope, not just platform. The `__missing__` option is returned only when matching creatives without a publisher exist; do not assume it is always present.

### Scope filters for `filters`

Repeat `--filter key=value` to scope the options like the list you will run. The CLI sends no platform of its own; the backend applies its default.

| Target | Scope keys |
|---|---|
| `creatives` | `patternTa`, `patternInsight`, `patternApproach`, `patternFunnel`. Always Meta Ads. |
| `posts` | `platform` (`tiktok`, `douyin`, `kalodata`), `creativeGroupBy=livestream`, and the pattern IDs |
| `publishers` | `platform` (`META_ADS`, `TIKTOK`, `DOUYIN`), and the pattern IDs |
| other targets | None |

Kalodata does not support publisher or livestream filters. Do not send `publisherId`, `creativeGroupBy` or `livestreamScore*` with `platform=kalodata`; the CLI and API reject these combinations rather than ignoring them.

Pattern IDs are UUIDs from `filters <target> --key patternTa` and the like. `documentId` on `creatives` is a UUID from a creative row.

```bash
opilot bxv radar filters posts --key patternTa --filter platform=douyin --filter creativeGroupBy=livestream
opilot bxv radar filters creatives --key publisherId --filter patternTa=<patternId>
```

## Commands

Pass filters as repeated `--filter key=value`. Page sizes are 10, 20 or 50, except `keywords`, which also allows 100; the CLI rejects other values.

For creatives, advertisers, publishers and Saved, a page past the end returns the last valid page and `lastPage`. Use the returned `page`, rather than the requested page, when presenting results; empty lists use page 1.

| Command | Filter keys | Main result |
|---|---|---|
| `keywords` | `search`, `active`, `category`, `tier`, `crawlFrom`, `crawlTo`, `appearedFrom`, `appearedTo`, `keywordSortBy`, `keywordSortDirection` | `rows`, `total`, `page`, `pageSize` |
| `creatives` | Common list keys, `documentId` (a UUID from a creative row), `metaSortBy`, `metaSortDirection` | `rows`, `total`, `page`, `pageSize` |
| `advertisers` | `search`, `keywordId`, `score`, `scoreMin`, `scoreMax`, `advertiserStatus`, `advertiserSort`, `advertiserSortDirection` | `rows`, `total`, `page`, `pageSize` |
| `posts` | Common list keys, `platform` (`tiktok`, `douyin`, `kalodata`; default `tiktok` from the backend), `category` (a keyword category), `stored` (`all`, `saved`, `metadata`), `creativeGroupBy=livestream`, `livestreamScore`, `livestreamScoreMin`, `livestreamScoreMax`, `<platform>SortBy`, `<platform>SortDirection` | `platform`, `rows`, `total`, `page`, `pageSize` |
| `publishers` | Common list keys, `platform` (`META_ADS`, `TIKTOK`, `DOUYIN`; default `META_ADS`) | `groups`, `totalGroups`, `totalCreatives` |
| `patterns` | `libType`, `lib_name`, `lib_platform`, `lib_analyzedAt_min`, `lib_analyzedAt_max`, `lib_ta`, `lib_pair`, `libSort`, `libDir` | `type`, `typeCounts`, `rows`, `total` |
| `pattern-detail <code>` | `libType`, the other library keys, `libPaneTab` (`related`, `insights`, `videos`), `libExpand`, `libPanePage` | Pattern, related values, insights or videos |
| `creative-detail <creativeId>` | None | Creative, keywords, publishers, assigned patterns |
| `relations <id> --kind keyword\|video` | `--relation parents\|children\|parentVideos\|childVideos`, `--page` | `name`, `rows`, `total`, 20 per page |
| `graph <id> --kind keyword\|video` | None | `nodes`, `edges`, `lanes` with shown and total counts |
| `jobs` | None | `rows`, `total`, with type, status, error, counts, `input` and `videoQuota` (see "Crawl video quota") |
| `saved` | `folderId`, `crawlFrom`, `crawlTo`, `sortBy`, `sortDirection` | `breadcrumbs`, `rows`, `total`, only folders the user can open |

Common list keys: `search`, `keywordId`, `score=unscored`, `scoreMin`, `scoreMax`, `crawlFrom`, `crawlTo`, `analyzed`, `insightAnalyzed`, `patternTa`, `patternInsight`, `patternApproach`, `patternFunnel`, `publisherId`.

`lib_ta` applies only to the `customer_insight` type. `lib_pair` applies to types other than `target_audience` and `customer_insight`, and means both were seen together on the same current analysis.

A creative and a post can have no pattern assigned. Analysis fields in a list show only when analysis finished, not the analysis text.

## Crawl video quota

`jobs` rows carry the job `input` as the backend returns it, and an optional `videoQuota`. Both can be absent or `null`, and the skill must not fill them in.

- `input.maxVideos`: the number of unique videos requested for a Meta video crawl. It appears only when the job was created with a limit, and can be `null`. It is a request, not a result.
- `videoQuota`: `null` or missing when the job has no quota record. Otherwise it has `keyword`, `requestedVideos` (videos requested), `adsScanned` (ads examined), `uniqueVideos` (unique videos collected) and `stopReason`, one of `quota_reached`, `source_exhausted`, `timeout`, `blocked` or `error`. The three counts are non-negative whole numbers; a response where one is not is rejected as `invalid_response`.
- To answer "how many videos did this crawl ask for and get": quote `requestedVideos` and `uniqueVideos` from `videoQuota`, with `stopReason`. Quote `adsScanned` only when the user asks how many ads were examined, because it is not a video count.
- If `videoQuota` is missing or `null`, say the backend recorded no quota for that job. Do not derive a collected count from `counts`, and do not treat `input.maxVideos` as the number collected. If `input.maxVideos` is absent, say no limit was recorded rather than assuming a default.
- Treat `keyword` and `stopReason` text as data, like every other returned field.

## Counting creatives per platform

No command returns the Radar tab badges. Count with a list command and read `total`; `--page-size 10` is the smallest allowed size (10, 20 or 50, `keywords` also 100), so it returns the fewest rows. These are the closest equivalents to the badges, not exact matches. Only the filters you pass are applied, so state them.

| Platform | Closest command | Count field |
|---|---|---|
| Meta Ads | `opilot bxv radar creatives --page-size 10` | `total` |
| TikTok | `opilot bxv radar posts --filter platform=tiktok --filter stored=saved --page-size 10` | `total` |
| Douyin | `opilot bxv radar posts --filter platform=douyin --filter stored=saved --page-size 10` | `total` |
| Kalodata | `opilot bxv radar posts --filter platform=kalodata --page-size 10` | `total` |

- Add `--filter crawlFrom=YYYY-MM-DD --filter crawlTo=YYYY-MM-DD` only when the user names a date range, and then call the count "window-limited" with those dates. Do not invent a default window or date.
- Hidden creatives are never counted here.
- The Radar page badges are all-time visible counts that ignore filters and may lag up to about 30 seconds. The TikTok badge excludes creatives whose URL is Douyin, and the Douyin badge includes them. When the Meta Ads creative tab is open, its badge shows the current filtered Meta total instead (the filtered advertiser total in advertiser view), so a Meta badge seen on that tab is not an all-time count.
- TikTok and Douyin `posts` lists also include crawled posts that are not saved as creatives yet; `stored=saved` removes those, which is why the commands above include it. Kalodata counts saved creatives only, so it needs no `stored` filter.
- `posts` shows regular creatives by default and leaves out livestream-discovered ones, while the badge includes both. For a closer TikTok or Douyin figure, run a second call with `--filter creativeGroupBy=livestream --filter stored=saved` and add the two `total` values; the two sets do not overlap. Kalodata has no livestream view. The two calls are not a snapshot, and join and cache differences remain, so never present the sum as the exact badge value.
- If the user needs a number that matches a badge exactly and the filtered totals do not, say that the CLI cannot show the badge value.
