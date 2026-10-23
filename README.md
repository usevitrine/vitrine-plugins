# Vitrine plugins

Site plugins for [Vitrine](https://github.com/usevitrine/vitrine), a self-hosted media library with a built-in
downloader. This repository is also Vitrine's default plugin index (`index.json`): install and update plugins in
Vitrine under Settings > Sites. You can also install a plugin's `.yaml` file there with "Install from file".

Some sites here host adult content. Plugins only describe how to talk to a site; respect each site's rules and the
law where you live.

| Plugin | Site | Engine | Needs an account |
|---|---|---|---|
| [`baraag`](plugins/baraag.yaml) | baraag.net | mastodon | no (an access token is optional) |
| [`danbooru`](plugins/danbooru.yaml) | danbooru.donmai.us | danbooru | no (optional: user name + API key) |
| [`realbooru`](plugins/realbooru.yaml) | realbooru.com | gelbooru-v02-html | no |
| [`rule34`](plugins/rule34.yaml) | rule34.xxx | gelbooru-v02 | yes: API key + user ID |
| [`rule34hentai`](plugins/rule34hentai.yaml) | rule34hentai.net | shimmie2 | no (optional login) |
| [`e-hentai`](plugins/e-hentai.yaml) | e-hentai.org | ehentai (galleries as comics) | no (optional login) |
| [`exhentai`](plugins/exhentai.yaml) | exhentai.org | ehentai (galleries as comics) | yes (an e-hentai login with ExHentai access) |
| [`safebooru`](plugins/safebooru.yaml) | safebooru.org | gelbooru-v02 | no (also built into Vitrine) |
| [`safebooru-donmai`](plugins/safebooru-donmai.yaml) | safebooru.donmai.us | danbooru | no (optional: user name + API key) |

## Site notes

What each site does (APIs, rate limits, quirks): [`docs/sites.md`](docs/sites.md).

## Writing a plugin

When you add a plugin or change its `version`, update its entry in `index.json` too (Vitrine offers an update when
the index has a newer version than the one installed).


A plugin is a manifest naming one of Vitrine's built-in engines plus the site's details (API address, the domains it
may contact, rate limits, how its ratings map to Vitrine's). See Vitrine's built-in example,
`internal/plugin/builtin/safebooru.yaml`. Sites that need more than a manifest will get sandboxed JavaScript plugins.

License: MIT.
