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

## baraag (`plugins/baraag.yaml`, engine `mastodon`)
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
