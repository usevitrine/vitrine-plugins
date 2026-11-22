# Site notes

What each plugin's site does, as found while writing the plugins (dates are when it was checked).

## rule34 (`plugins/rule34.yaml`, engine `gelbooru-v02`)
- API: `https://api.rule34.xxx/index.php?page=dapi&s=post&q=index&tags=...&pid=<page>&limit=100`; needs `api_key` +
  `user_id` (a free account). One response has tags, md5, source and the file URL for up to 100 posts. Query syntax:
  OR groups `( a ~ b )`, `-tag` excludes. API help: https://rule34.xxx/index.php?page=help&topic=dapi
- Request URLs carry the api_key - never log or print them unfiltered.
- Tag types: the JSON API with `fields=tag_info` (one extra request per page; the XML API has no types, the JSON one
  has no upload time or total). No batch tag lookup (`names=` is ignored) and no OR of IDs.
- Rate limits (2026-10-14): back-to-back lookups at 1 request/s nonstop got `429 Too Many Requests` after about two
  minutes; about one request a second with pauses is fine.
- Files (2026-10-15): one at a time ~18 MB/s, three at once ~50 MB/s, so the plugin uses `connections: 3`.
  `max_connections: 6` (2026-10-28, plugin 1.4.0, needs Vitrine 1.0.6: `min_vitrine` in the index): through a VPN
  exit far from the video server single connections crawl at 100-300 KB/s, so Vitrine tries more and keeps them while
  they add up.
- Posts often carry a source link (usually the artist's post elsewhere).
- Paging depth (2026-10-27, the web list pages via FlareSolverr): offsets from somewhere between 75k and 100k posts
  answer "Unable to search this deep in temporarily." (no posts). The engine never pages deep - always `pid=0` plus
  `id:<N` - and `id:<N` pages stay fine at any depth (checked down to id 5,000,000), so searches reach their oldest
  post. Favorites pages page by `pid` (favorites.go), so a favorites list past the cap would stop there.
- Files are fetched with `?<post id>` (`file_query`), as rule34's own pages link them (2026-10-21): through a US exit,
  Cloudflare's Washington cache (IAD) had broken copies of some files under the bare URL the API gives - "HIT", then
  a crawl stopping at the same byte every time (London's cache was fine); the `?<id>` URL is cached apart, whole.

## realbooru (`plugins/realbooru.yaml`, engine `gelbooru-v02-html`)
- Its API answers "API offline because apparently it is broken ... shut off indefinitely" (2026-10-15), so the plugin
  reads the web pages with Vitrine's `gelbooru-v02-html` engine.
- List pages: `page=post&s=list&tags=...&pid=0`, 42 posts per page, `pid` = post offset; the site's `id:<N` search
  works for resuming. The thumbnail is named `thumbnail_<md5>.jpg`, so seen posts are skipped without opening them.
- Post pages: tag classes `tag-type-general`, `artist`, `character`, `copyright`, `model` (the performers),
  `metadata` -> namespaces creator, character, series, model, meta. Videos have mp4 and webm `<source>`s (mp4 first;
  some mp4s are missing or empty); the upload date reads "Posted at Oct, 14 2026" (day only) - the file's `Last-Modified` header has the moment (always
  the same day; checked 2019-2026, images and videos), which Vitrine uses when it downloads the file; no rating (the manifest
  maps it to explicit).
- Hotlink protection: file URLs redirect to `hotlink.php` (an HTML page) unless the request carries a realbooru
  Referer - the manifest sets `referer:`.
- Account (optional, 1.1.0, 2026-10-21): the Gelbooru 0.2 login form (`page=account&s=login&code=00`, fields `user`,
  `pass`; cookies `user_id` + `pass_hash`) - Vitrine's `login: gelbooru`; `fav:me` = the signed-in user's favorites
  (`page=favorites&s=view&id=<user_id>`). Anyone's: `fav:<user ID>` without an account. Not live-tested with an account yet.
- Speed (2026-10-12): about 1 MB/s per IP; extra connections get throttled to almost nothing, so `connections: 1`.
  Re-measured 2026-10-21 through a US VPN exit: one connection 1.8-4.5 MB/s (busier or not), 2 connections ~half the
  total, 4: three of them at ~0.09 MB/s; from a UK home line ~0.6 MB/s. A post page alongside a download doesn't
  slow it (Vitrine opens the next post's page meanwhile).
  Videos are big (60-390 MB seen).

## rule34hentai (`plugins/rule34hentai.yaml`, engine `shimmie2`)
- Shimmie2. Its APIs are off (2026-10-16: `/api/danbooru/find_posts` and `/graphql` answer 404), so the plugin reads
  the web pages with Vitrine's `shimmie2` engine.
- Behind **Cloudflare**: from some connections every page (even the API paths) answers a challenge
  (`cf-mitigated: challenge`), from others plain requests get the pages (2026-10-16). When challenged, set up
  FlareSolverr in Vitrine (Settings > Sites > Cloudflare).
- List pages: `/post/list/<tags>/<page>`, 63 posts a page; tags joined with `+` (a `%20` join finds nothing); an empty
  result is a 404 page. **Without `order=id_desc` the site mixes older posts in** (also with `id<N`), so the engine
  always adds it. `id<N` works for resuming. Each thumbnail link has `data-post-id`, `data-tags` (lower-case),
  `data-width/height`, `data-mime`; the thumbnail is `/_thumbs/<md5>/thumb.jpg`, so seen posts are skipped unopened.
- Post pages: the file in the Download form (`/_images/<md5>/<id> - <tags>.<ext>`), tags as `a.tag_name` (no tag
  types), upload time `<time datetime>` in the Uploader row, Source and Rating rows (most posts "Unrated" - the
  manifest maps that to explicit).
- Login: the form at `/user_admin/login` (fields `user`, `pass`); the session is the `shm_user`/`shm_session` cookies.
  The plugin uses `auth.login: shimmie2`, so the user name and password never go into request URLs.

## baraag (retired 2026-11-20: `retired/baraag.yaml`, engine `mastodon`)
- **Gone**: baraag.net stopped resolving around 2026-11-19 (its domain fails on every DNS server tried). The
  plugin is out of the index, kept in `retired/` (with its index entry) in case the site comes back.
- Mastodon 4.7.2 (a fork, github.com/baraag-net/mastodon). API rate limit 300 requests / 5 min per IP; up to 4
  files per post; files on `media.baraag.net` (images, `gifv` = mp4 animations, videos).
- `timelines_access` (2026-10-16): hashtag feeds `local: public, remote: authenticated` - guests get a 422 "requires
  an authenticated user" unless they ask `local=true` (baraag users' posts only); profiles work as a guest.
- Hashtag timelines take `all[]` (and), `any[]` (or), `none[]` (not) - all three work. Profiles:
  `/api/v1/accounts/lookup?acct=<name>`, then `/accounts/<id>/statuses?only_media=true&exclude_reblogs=true`
  (`tagged=` filters by one hashtag). Paging with `max_id` (no Link header with these queries).
- Searches in Vitrine: `#tag` (hashtags; a plain word works too), `@profile`, `-#tag` (without), `~#tag` (either);
  the plugin's `search.hint` shows that in the new-download box. A tag's "New download" in Vitrine writes a creator
  as `@name` and other tags as `#tag` (the Mastodon engine's `search.tag_query` defaults; nothing to set here).
- The plugin: each file = one Vitrine post (ID status*10+n), the hashtags (lower-cased) + the author's `acct` as a
  creator tag. Optional access token (Preferences > Development > New application, scope `read`) - adds other
  servers' posts to hashtag searches.

## danbooru and safebooru-donmai (`plugins/danbooru.yaml`, `plugins/safebooru-donmai.yaml`, engine `danbooru`)

- JSON API (`/posts.json`, `/posts/<id>.json`, `/counts/posts.json`); files on cdn.donmai.us. Reads are limited to
  about 10 a second; the plugins ask 2. safebooru.donmai.us is the same site showing only posts rated general.
- Files: 3 at once, up to 6 where more add up (`max_connections`, 2026-10-28, plugins 1.1.0, need Vitrine 1.0.6 -
  `min_vitrine` in the index): cdn.donmai.us is a CDN, and the request limit covers the API, not files.
- No account: two tags per search (more is refused with 422), and some posts come without a file URL (banned, or
  for members only) - the engine passes those over. An account: user name + API key (My Account > API Key; the
  posts:index and posts:show permissions are enough) in Settings > Sites.
- Searches: `blue_sky -cloud`, `~a ~b` (either), `fav:<name>` (someone's favorites; `fav:me` = yours with an account),
  plus the site's metatags (`rating:g`, `score:>10`, `order:` - ordering other than by ID breaks resuming, avoid it).
- Checked live 2026-10-20 on safebooru.donmai.us (search, next page, lookup); danbooru.donmai.us not yet.

## e-hentai and exhentai (`plugins/e-hentai.yaml`, `plugins/exhentai.yaml`, engine `ehentai`)
- Galleries come in as comics (Vitrine's Comics page), never as library files. Search with the site's own syntax
  (`artist:"some artist$"`, `female:"glasses$"`, words), or paste a gallery's link to download that one comic
  (subscribed, newer versions update it).
- Login: optional on e-hentai.org, needed on exhentai.org - the forum's user name and password, or (when the login
  asks for a captcha) the `ipb_member_id` and `ipb_pass_hash` cookies from a logged-in browser. ExHentai's blank
  "sad panda" page = the account has no ExHentai access yet.
- Gentle by default: a page a second, one picture at a time. The site's image limit stops a download until it
  refills (logged in, the account's own limit applies); it carries on from the page it stopped at.
- e-hentai blocks the United Kingdom (HTTP 451, Online Safety Act): reach it from elsewhere (a VPN).

## yandere, konachan, konachan-net and sakugabooru (`plugins/yandere.yaml`, `konachan.yaml`, `konachan-net.yaml`, `sakugabooru.yaml`, engine `moebooru`)

- Moebooru's JSON API (`/post.json?api_version=2&include_tags=1`: a page of posts plus every tag's type); the total
  from `/post.xml`'s `count`. Up to 1000 posts a page (the plugins ask 200). No documented rate limit; the plugins
  ask 2 requests a second, 2 files at once (up to 4).
- Tag types: artist, circle (both the creator), copyright (series), character, faults and meta (meta), general;
  konachan's "style" stays a plain tag.
- Searches: `blue_sky -cloud`, `~a ~b` (either), `fav:<name>` (that user's favorites: the site's `vote:3:<name>`).
  No sign-in (Moebooru's login is a salted password hash, not an API key), so no `fav:me`. Ratings `s`/`q`/`e`.
- yande.re's files are on files.yande.re. konachan.net is konachan.com's safe-for-work side (every post rated s);
  from some countries (the UK, 2026-11-17) konachan.com answers every request with a redirect to konachan.net -
  the plugin doesn't follow it (another site): use konachan-net there, or reach konachan.com from elsewhere.
- sakugabooru: animation clips, nearly all mp4 (a few images); files on www.sakugabooru.com.
- Checked live 2026-11-17 (search, the next page, a lookup) on konachan.net; yande.re, konachan.com (from a non-UK
  address) and sakugabooru answered the same API.

## e621 and e926 (`plugins/e621.yaml`, `plugins/e926.yaml`, engine `e621`)

- JSON API (`/posts.json`, `/posts/<id>.json`), up to 320 posts a page, resumed with `page=b<ID>`. Every post has
  its tags by category: artist (creator), contributor, copyright (series), character, species, meta, lore, general
  (invalid ones stay plain). Files on static1.e621.net / static1.e926.net.
- The site allows 2 requests a second at most and asks for 1 (the plugins ask 1); it wants a User-Agent naming the
  program (Vitrine's does - a browser's gets Cloudflare's check).
- No count of a search: a one-tag search's total is that tag's post count (`/tags.json`); other searches, and every
  search on e926 (its tag counts include the posts it doesn't show), have none.
- Account optional: user name + API key (Account > Manage API Access), sent in the query. Without one some posts
  come without a file URL (the site's default blacklist for guests); they're passed over. A wrong key is a 401.
- Searches: `blue_sky -cloud`, `~a ~b` (either), `fav:<name>` (`fav:me` with an account), the site's metatags
  (`rating:s`, `score:>10`; `order:` other than by ID breaks resuming - avoid it).
- e926.net shows only posts rated safe. Checked live 2026-11-17 on e926.net (search, the next page, a lookup).

## redgifs (`plugins/redgifs.yaml`, engine `redgifs`)

- API v2 at api.redgifs.com, with the temporary token anyone gets from `/v2/auth/temporary` (a JWT, sent as a
  bearer token to the API only; Vitrine keeps it a day and fetches another when refused). Files on media.redgifs.com
  (no token needed): `urls.hd`, `urls.sd` as the fallback.
- Searches: tags (`/v2/gifs/search?tags=a,b&order=latest`: clips with all of them; case doesn't matter; no `-tag`, no
  OR), `@user` (`/v2/users/<name>/search?order=new`), or a clip's link (`/watch/<id>`, one clip). A tag search gives
  one kind at a time (`type=g` videos, the default, or `type=i` pictures - `type:images` in the search); a user's gives
  both. 80 a page; a tag search stops after 10,000 clips (page 126 of 80 answers 500), a user's goes to the end.
- The tag search is loose: a tag the site doesn't know is ignored (a search for it gives everything) and known ones
  sometimes match clips without them - Vitrine keeps only clips with every tag, and says when none has them all.
- Tags come as words ("Big Smile"): stored as `big_smile`. The poster's user name is the creator tag.
- Clips have word IDs; Vitrine numbers them by upload time (see the engine's notes in the app's docs), so a clip's
  `id:N` or a lookup by number doesn't work - its link does.
- Checked live 2026-11-17 (a tag search, resuming after a restart, a clip's link).

## xbooru and hypnohub (`plugins/xbooru.yaml`, `plugins/hypnohub.yaml`, engine `gelbooru-v02`)
- Gelbooru 0.2 with the dapi on, no account needed (2026-11-18). Both answer "Access Restricted" from some countries
  (the UK); fine from elsewhere.
- OR searches `( a ~ b )` work on both. Ratings `s`/`q`/`e`.
- Tag types: xbooru's JSON API gives `fields=tag_info` (artist, character, copyright, metadata, tag - the rule34
  names); hypnohub's has none, so its tags come without namespaces.
- Files: xbooru on `img.xbooru.com`, hypnohub on the site itself.

## paheal (`plugins/paheal.yaml`, engine `shimmie2` with `api: danbooru`)
- rule34.paheal.net: Shimmie2 with its theme's own list markup (the thumbnail's data is on a `div.shm-thumb`, not on
  the link the `shimmie2` engine reads), but **its Danbooru-style API is on** (2026-11-18):
  `/api/danbooru/find_posts?tags=...&limit=N` (at most 100) and `?id=N` for one post. Each post has id, md5,
  `file_url` (no extension - the type comes from the file), width/height, rating (`?` on almost all: unrated),
  `date` (`2026-11-18 00:53:59`, UTC), tags (with capitals; searches ignore case) and source.
- `id<N` and `order=id_desc` work in the API's tags, as on the pages.
- **HTML entities in the XML** (`&oacute;` in tags): the engine decodes them (Go's XML decoder only knows XML's five).
- Files on `r34i.paheal-cdn.net`, no referer needed. Many animated posts (mp4, webm, gif).
- No account in the plugin (searches work without one).

## aibooru (`plugins/aibooru.yaml`, engine `danbooru`)
- aibooru.online: Danbooru for AI-generated pictures (2026-11-18). Tags have an extra type, `tag_string_model` (the AI
  model), read by the danbooru engine as the `model` type -> the `model:` namespace.
- Files on `cdn.aibooru.download` (a second domain in `domains`).
- **Cloudflare through the VPN**: Vitrine's own requests get challenged even with a clearance while FlareSolverr's
  browser is let through, so every request falls back to the browser's page - Chrome's JSON viewer (`<pre>` around
  the JSON), which Vitrine unwraps. It works, slowly; FlareSolverr is needed there. Without the VPN the API answers
  plain requests.

## pawoo (`plugins/pawoo.yaml`, engine `mastodon`)
- Mastodon 4.1 (2026-11-18). Unlike baraag, hashtag timelines work as a guest with other servers' posts included.
  Files (others' too, cached) on `img.pawoo.net`. API rate limit 300 / 5 min.

## gelbooru (`plugins/gelbooru.yaml`, engine `gelbooru-v02`, tag types `tag_api`)
- gelbooru.com runs a newer Gelbooru than the 0.2 sites (2026-11-18): the API needs an account's API key and user ID
  (401 without); its XML has each post's fields as elements, not attributes; its JSON has no `fields=tag_info`, so
  tag types come from the tag API (`s=tag&names=...`, types as numbers: 0 general, 1 artist, 3 copyright,
  4 character, 5 metadata, 6 deprecated).
- An account may make 10 API requests a second (its options page says so; more on request); the plugin uses all 10, one every 100 ms for all its API requests together (files don't count).
- The image servers (`img*.gelbooru.com`) redirect a file to its post page without a Referer.
- Not behind Cloudflare through the VPN (Ireland).

## reddit (`plugins/reddit.yaml`, engine `reddit`)
- Without OAuth every Reddit API address answers 403 "Blocked" through the VPN (2026-11-18); the token endpoint
  answers. So the plugin needs the user's own app (reddit.com/prefs/apps, a script app): its client ID and secret,
  and optionally the account's username and password (not with two-factor sign-in).
- 100 API requests a minute with OAuth: the plugin asks once a second. Files: i.redd.it, v.redd.it (videos, the sound
  apart - joined), preview.redd.it, and media.redgifs.com for the RedGIFs clips posts link.
- `site: reddit.com` is set (the API's host is oauth.reddit.com).
- **Not in index.json** : Reddit's Responsible Builder Policy needs an approved app; the project applied for one
  (2026-11-18), and Reddit answered (2026-11-22) that it has stopped accepting API access requests altogether -
  apps must run on its Developer Platform, inside Reddit. Shelved: the file stays here in case that changes.

## Tried, not added
- **tbib.org** (Gelbooru 0.2, a mirror of other boorus; no tag types): through the VPN (Ireland) Cloudflare answers
  "Attention Required!" - a block, not a challenge, so FlareSolverr can't help (2026-11-18). Works from other
  connections.
