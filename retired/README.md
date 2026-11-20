# Retired plugins

Plugins for sites that went away, kept here in case they come back. They aren't in `index.json`, so Vitrine
doesn't offer them (a copy someone installed before keeps working, or not, with the site).

To bring one back: move its file to `plugins/`, add its entry below to `index.json` (in A-to-Z order), raise its
`version` if anything in it changes, and add its row to the README's table.

## baraag (retired 2026-11-20)

baraag.net stopped resolving around 2026-11-19: its domain fails on Cloudflare's and Google's DNS ("server failed"),
from anywhere. Engine `mastodon` (still in Vitrine: Pawoo uses it). Its notes: `docs/sites.md`, "baraag".

README row:

| [`baraag`](plugins/baraag.yaml) | baraag.net | mastodon | no (an access token is optional) |

Index entry:

```json
{
  "id": "baraag",
  "name": "baraag.net",
  "site": "baraag.net",
  "version": "1.1.0",
  "file": "plugins/baraag.yaml",
  "description": "Mastodon server for art: #hashtags (#art #furry, ~#either, -#without) and @profiles. Files and hashtags, the author as the creator. An access token is optional.",
  "kind": "social",
  "nsfw": true
}
```
