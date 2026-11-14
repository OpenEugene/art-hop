---
name: wikidown
description: Use whenever the user asks to read, write, search, rename, or reorganize pages in the project's Wikidown wiki at /docs. Triggers include phrases like "add a wiki page", "update the docs", "what does the wiki say about X", and any task that touches /docs/*.md.
---

# Wikidown skill

This repo's documentation lives in `/docs` and is a **Wikidown** wiki — a
structured folder of markdown pages with `.order` navigation files. Edit it
through the `wikidown` CLI, never by writing the `.md` files directly. If the
CLI can't run, follow "Last resort: no CLI" below.

Run commands from the repo root; the default wiki root is `./docs`
(override with `--root <path>`).

## Format rules

- **Link path** — title form, hyphens for spaces: `/Getting-Started/Format`.
  This is how the CLI addresses pages (`--path /Venues/Pinebox`).
- **File on disk** — `Getting-Started/Format.md`. Subpages of `/Parent` live
  in a `Parent/` folder beside `Parent.md`.
- **Order** — each folder's `.order` file controls navigation order. Page
  writes update it automatically; rewrite explicitly with `wikidown reorder`.
- **Internal links (body)** — relative, with `.md`, adjusted for depth:
  `[Format](../Getting-Started/Format.md)`. Images: `![map](../.attachments/map.png)`.
  GitHub renders `/docs/*.md` directly and resolves an absolute path like
  `/Getting-Started/Format` against the repo root, not the wiki root — those
  links 404 on github.com. This is only about links written inside page
  bodies; `--path` arguments still use the title-path form.
- **Page structure** — start with `# Title`, then a one-sentence summary,
  then content under H2/H3 headings.

## Command cheat sheet

| Intent                  | Command                                                          |
| ----------------------- | ---------------------------------------------------------------- |
| What pages exist?       | `wikidown walk` (everything) or `wikidown list [--path /P]`      |
| Read a page             | `wikidown read --path /P`                                        |
| Read one section        | `wikidown read --path /P --section "Heading"`                    |
| Create a page           | `wikidown new --path /P [--title T] [--file F \| --stdin]`        |
| Change part of a page   | `wikidown edit --path /P --old <text> --new <text> [--all]`      |
| Multi-line change       | `wikidown edit --path /P --old-file F1 --new-file F2`            |
| Delete text             | `wikidown edit --path /P --old <text> --delete`                  |
| Rewrite one section     | `wikidown write-section --path /P --section "Heading" --file F [--create]` |
| Add to the end          | `wikidown append --path /P [--after "Heading"] --file F`         |
| Rewrite a whole page    | `wikidown write --path /P --file F`                              |
| Find a topic            | `wikidown search --query <text>`                                 |
| Rename or move          | `wikidown move --from /A --to /B [--dry-run]`                    |
| Delete (with subpages)  | `wikidown delete --path /P --recursive`                          |
| Re-sort a folder        | `wikidown reorder --folder /P --names a,b,c`                     |
| Validate links          | `wikidown check-links`                                           |

For multi-line content, write it to a temp file in the scratchpad (not in
`/docs`) and pass `--file`, or pipe it with `--stdin` using a heredoc.

## Workflow

1. **Orient.** Run `wikidown walk` once at the start so you know what exists.
2. **Search first.** `wikidown search` before creating a page — you may just
   need to update an existing one.
3. **Read before overwriting.** `wikidown read` first; preserve voice and
   structure. On a long page, read just the section you need with
   `--section "Heading"` (case-insensitive, `#`s optional; a `##` section
   includes its `###` children). A miss lists the page's headings.
4. **Patch, don't rewrite.** Prefer `wikidown edit` for any change smaller
   than a full rewrite — one line, one bullet, one table row, a renamed
   heading. Pass `--old` exactly as it appears on the page and enough of it
   to be unique — the command refuses ambiguous matches and says how many
   times the text matched. To rewrite one whole section, `write-section` —
   pass the new body without the heading line; the heading stays and
   everything under it (including `###` children) is replaced. To add at the
   end of a page or of one section, `append` (with `--after`). Use `write`
   only for deliberate full rewrites and `new` for new pages.
5. **Cross-link.** When you create or rename a page, update inbound links on
   sibling pages with `wikidown edit`.
6. **Order intentionally.** When adding a top-level concept, `wikidown
   reorder` so the new page lands where it makes sense in navigation.
7. **Moves rewrite links automatically.** `wikidown move` rewrites inbound
   links and the moved page's own relative links/images for their new depth,
   and reports what it changed. Use `--dry-run` to preview.
8. **Verify.** Run `wikidown check-links` after structural changes.

## Install

```sh
dotnet tool install -g Wikidown.Cli
# or, without .NET:
curl -fsSL https://wikidown.org/install.sh | sh   # Windows: irm https://wikidown.org/install.ps1 | iex
```

## Exporting

- `wikidown export-html --output <dir> [--base-url /prefix] [--clean]`
  renders the wiki to a static HTML site.
- `wikidown export-pdf --output <path> [--from /P] [--title T]` combines the
  whole wiki (or a subtree) into one linked PDF — cover page, table of
  contents, bookmarks matching the nav hierarchy, and in-PDF jumps for
  internal links. Use it whenever asked for a PDF or printable copy.

## Last resort: no CLI

If the CLI cannot run (no .NET and an execution policy blocks the
self-contained binary), wiki edits may be made with plain file edits — but
only by maintaining every invariant the CLI normally handles:

1. **Breadcrumb line.** Every page's *first line* is a breadcrumb, marked
   with an HTML comment:

   ```markdown
   [Home](../Home.md) / [Parent Title](../Parent.md) / Page Title <!-- wikidown:breadcrumb -->
   ```

   Lead with a link to `/Home` if the wiki has one, then each ancestor as a
   relative `.md` link (depth-adjusted from the page's own folder), then the
   page's own title unlinked, then the marker comment. A page at the wiki
   root with no `/Home` has no breadcrumb line. When you create or move a
   page, write/regenerate this line; never duplicate it.

2. **`.order` bookkeeping.** Each folder's `.order` lists page base names
   (no `.md`), one per line, LF endings. On create: append the new page's
   base name. On delete: remove it. On move: remove from the old folder's
   file, add to the new one's.

3. **Create.** The parent page must exist and its body must link the new
   child (`[B](A/B.md)`), otherwise the subtree is invisible to readers.

4. **Move/rename.** Move the `.md` file *and* its same-named subpage folder;
   rewrite every inbound link; re-adjust the moved page's own relative
   links/images if its depth changed; regenerate breadcrumbs of the moved
   page and its descendants; update both `.order` files.

5. **Delete.** Remove the page, its subpage folder (if intended), its
   `.order` entry, and every inbound link to it.

Treat this as a degraded mode and say in your summary that edits were made
manually, so the user knows to run `wikidown check-links`.

## Don'ts

- Don't write `/docs/*.md` with file-edit tools while the CLI works.
- Don't `wikidown write` a whole page to change one line — use `edit`.
- Don't put temp content files inside `/docs`.
- Don't link to GitHub blob URLs from inside the wiki, and don't use
  absolute `/Title/Path` links in page bodies — use relative `.md` links.
- Don't rename without checking inbound references first.
- Don't write one-off chat notes into the wiki.
