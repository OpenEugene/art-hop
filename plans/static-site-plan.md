# thearthop.org static site plan

Status: draft, 2026-09-13. Not part of the published wiki.

## Goal

Rebuild warehousedistrictarthop.com (currently Squarespace) as a static
site on GitHub Pages at thearthop.org, running in parallel until cutover.
Keep the look and voice. Keep the full name "Warehouse District Art Hop"
in all copy. All page content lives in the Wikidown wiki under `docs/`.
Interactive features live in a separate Oqtane companion site; this site
only links to it through configured endpoints.

## What the current site is

- Squarespace. Pages: Home, Become a Member, Join Our Mailing List,
  current Art Hop map, Past Art Hop Features, cart.
- Home: huge "THE ART HOP" hero, "September art hop map here" link, photo
  highlights grid with numbered captions, one featured artist spotlight,
  "May's participating venues" two-column list (venue name left, 3 to 5
  sentence blurb with external links right), email signup, footer with
  Instagram / Email / Facebook and the membership pitch.
- Map page: a single JPG per month.
- Past features: long-form articles by month, several photos each.
- Membership: six recurring Squarespace subscriptions. Founding Fan $3,
  Big Believer $5, Super Supporter $10, Fiscal Friend $15, Artists'
  Advocate $20, Howie's Hero $40.
- Style: off-white background #FDFDFD, ink #1D1D1D, one accent orange
  #FF340A used for headings, links and buttons, light gray #F2F2F2 bands.
  Fonts: Staatliches for headings, Dosis for body, Newsreader for nav and
  buttons. All three are on Google Fonts.

## Build and hosting

- GitHub Pages, "Deploy from a branch", branch `main`, folder `/docs`.
  GitHub's own Jekyll builder renders the site. Nothing runs locally;
  push and wait a minute.
- `wikidown pages --title "Warehouse District Art Hop"` scaffolds
  `_config.yml`, `index.html`, `_layouts/wikidown.html`,
  `_includes/nav-tree.html`, `assets/wikidown.css`, and keeps
  `_data/navigation.yml` regenerated. We then replace the theme files.
- GitHub's builder ships the plugins the scaffold relies on
  (optional-front-matter, relative-links, titles-from-headings,
  default-layout) plus core Jekyll features we add on: `_data/*.json`,
  `site.pages` with content, Liquid `where` and `where_exp`, per-path
  layout defaults in `_config.yml`.
- Custom domain: set thearthop.org in repo Settings > Pages. GitHub
  commits `docs/CNAME`. Enforce HTTPS once the certificate issues.
- DNS at the registrar: four A records for thearthop.org to GitHub Pages
  IPs (185.199.108.153, .109.153, .110.153, .111.153), CNAME www to
  `<account>.github.io`.
- Cutover later: GitHub Pages allows one custom domain per site, so
  warehousedistrictarthop.com becomes a registrar-level redirect to
  thearthop.org when Squarespace is retired.
- `wikidown export-html` compatibility is a nice-to-have, not a
  requirement. Sticking to standard Liquid keeps the door open; anything
  Jekyll-only we use (`where_exp`, `_data` loading, page content in
  templates, `_config.yml` layout defaults) would need exporter support
  before export-html could render this site. Not planned for v1.

## Content model

Wiki pages (edited only through `wiki_*` tools):

```
docs/
  Home.md                      hero tagline and intro copy
  Map.md                       thin page; layout injects the current map
  Venues.md                    index page, links every venue
  Venues/<Venue-Name>.md       one page per venue: the blurb shown in the
                               monthly listing plus links; grows over time
  Features.md                  index of past features
  Features/<YYYY-MM>-<Slug>.md long-form monthly feature articles
  Membership.md                pitch copy; buttons come from config
  Mailing-List.md              copy; form comes from config
  About.md                     contact and organizer info
  .attachments/maps/<YYYY-MM>.jpg
  .attachments/features/<YYYY-MM>-<slug>/*.jpg
  .attachments/highlights/*.jpg
```

Config and schedule (plain JSON, edited directly, not wiki pages):

```
docs/_data/site.json           endpoints and settings, see below
docs/_data/hops/<YYYY-MM>.json one file per Art Hop month
```

Why JSON and not front matter: Wikidown pages carry no front matter by
design (the breadcrumb is line one, the title is the first heading), and
the CLI and MCP tools would fight it. Jekyll reads `_data/*.json` natively,
JSON is strict, and the Oqtane side can consume the same files later.
`_data/hops/2026-09.json` is reachable as `site.data.hops["2026-09"]`.

`_data/hops/2026-09.json`:

```json
{
  "date": "2026-09-05",
  "time": "5 to 9 pm",
  "map": "/.attachments/maps/2026-09.jpg",
  "feature": "Features/2026-09-Oblivion-Gallery.md",
  "venues": [
    { "page": "Venues/Pinebox.md" },
    { "page": "Venues/Caffe-Pacori.md" },
    { "page": "Venues/Viking-Brewing.md", "note": "Kitchen closes at 8." }
  ]
}
```

A venue is on for a month when its page path is listed in that month's
file. Off is simply absence. The venue page stays in the wiki either way,
so its history and links are never lost. Each entry can carry a per-month
`note` or `featured` flag. `site.json` names the current month, so
publishing next month is: add a map image, add a hops file, change one
key, push.

The venue list include looks each entry up with
`site.pages | where: "path", entry.page | first`, splits the content on
the `<!-- wikidown:breadcrumb -->` marker to drop the breadcrumb, and
renders the rest with `markdownify`. The venue page's H1 is the venue
name and becomes the left column. Caveat: `markdownify` on another page's
content bypasses jekyll-relative-links, so venue blurbs should use
absolute site paths or external URLs for links, not `../Foo.md`.

`_data/site.json`:

```json
{
  "currentHop": "2026-09",
  "name": "Warehouse District Art Hop",
  "email": "warehousedistrictarthop@gmail.com",
  "social": { "instagram": "...", "facebook": "..." },
  "nav": [
    { "title": "Home", "url": "/" },
    { "title": "Map", "url": "/Map.html" },
    { "title": "Venues", "url": "/Venues.html" },
    { "title": "Past Features", "url": "/Features.html" },
    { "title": "Become a Member", "url": "/Membership.html" },
    { "title": "Mailing List", "url": "/Mailing-List.html" }
  ],
  "membership": {
    "tiers": [
      { "name": "Founding Fan", "price": 3, "url": "https://..." },
      { "name": "Big Believer", "price": 5, "url": "https://..." },
      { "name": "Super Supporter", "price": 10, "url": "https://..." },
      { "name": "Fiscal Friend", "price": 15, "url": "https://..." },
      { "name": "Artists' Advocate", "price": 20, "url": "https://..." },
      { "name": "Howie's Hero", "price": 40, "url": "https://..." }
    ]
  },
  "newsletter": {
    "provider": "brevo",
    "formAction": "https://...brevo form POST url...",
    "embedUrl": ""
  },
  "oqtane": {
    "baseUrl": "https://app.thearthop.org",
    "endpoints": {
      "artistSignup": "/signup",
      "venueLogin": "/venues"
    }
  }
}
```

Membership buttons stay as six styled links. Until Stripe exists, point
each `url` at the existing Squarespace product page so the parallel site
works on day one. When Stripe is ready, replace with Stripe Payment Links
(recurring, no code, one link per tier). Brevo: the plain HTML form posts
to Brevo's form action URL, which works from a static page; `embedUrl` is
the fallback if the iframe embed is preferred. Oqtane: the site only
renders links built from `baseUrl` + endpoint; nothing is fetched at
build time or in the browser in v1.

## Theme

- `docs/index.html`: standalone home page (`layout: none`, as scaffolded).
  Pulls the `/Home` page body the same way the venue include does.
  Sections: top nav from `site.data.site.nav`, hero, "Map" call to action
  from the current hop, highlights grid, featured page excerpt, this
  month's participating venues, mailing list form, membership strip,
  footer. Wikidown already resolves links to Home as `/`, so serving
  Home's content from `index.html` keeps breadcrumbs and nav consistent.
- `docs/_layouts/wikidown.html`: restyled for interior pages. Replace the
  wiki sidebar with the same top nav and footer as the home page. Keep
  breadcrumbs. Move the "Published with Wikidown" line to footer small
  print.
- `docs/_includes/`: `nav.html`, `footer.html`, `venue-list.html`,
  `membership.html`, `newsletter.html`, `hop-map.html`.
- `docs/assets/site.css`: palette and fonts above, Google Fonts link in
  the layouts. Type scale: hero heading very large Staatliches, section
  headings Staatliches in orange, body Dosis, nav and buttons Newsreader.
- `Map.md`: the layout injects the current month's map image, date and
  time from `site.data` when `page.path == "Map.md"`.
- `_config.yml`: `title`, `description`, `exclude` for `plans/` is not
  needed since plans live outside `docs/`. Add `keep_files` nothing.

## Phases

1. Theme: run `wikidown pages --title "Warehouse District Art Hop"`,
   then replace `index.html`, layout, includes, CSS. Add
   `_data/site.json` and the first `hops` file. Push, enable Pages on
   `/docs`, check the build at `<account>.github.io/art-hop`.
2. Content migration: venue pages, current map, membership and mailing
   list copy, About, and the five past features with their photos.
   Resize photos to about 1600px wide before adding. Photos are the
   largest risk for repo size; keep originals out of git.
3. Domain: add thearthop.org in Pages settings, point DNS, wait for the
   certificate, enforce HTTPS.
4. Wire endpoints: Stripe links, Brevo form, Oqtane URLs in `site.json`.
5. Cutover: redirect the old domain, retire Squarespace.

## Open questions

- Which GitHub account or org hosts the repo (affects the www CNAME and
  the preview URL before the domain is attached).
- Whether the highlights grid captions should be a wiki page or a JSON
  list under `_data`. Recommendation: a wiki page `Highlights.md` with an
  image and caption per item, rendered by an include.
- Photo licensing and who owns originals from the Squarespace site.
