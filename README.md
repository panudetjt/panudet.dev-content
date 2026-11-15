# panudet.dev-content

The build source for [panudet.dev](https://panudet.dev). The website repo reads this repository at
build time and prerenders every public page; it is not a runtime dependency.

## Layout

| Path | Contents |
|---|---|
| `posts/<slug>.md` | Published posts. Plaintext; safe to read and diff. |
| `drafts/<uuid>.enc` | Unpublished posts. AES-256-GCM ciphertext; no title or body in plaintext. |
| `images/<hash>-<name>.<ext>` | Post images referenced by markdown. |
| `assets/og/<slug>.png` | Generated Open Graph images (committed by CI). |

## Editing posts

Published posts are ordinary markdown files with a YAML frontmatter block:

```markdown
---
title: "A post"
slug: "a-post"
date: "2026-11-15T00:00:00.000Z"
updated: null
summary: "One sentence."
tags:
  - "zettel"
para: "projects"
zettel: "permanent"
lang: "th"
hero: "images/abc123-cover.webp"
---

## Heading

Body text.
```

`para` is one of `projects`, `areas`, `resources`, `archives`; `zettel` is one of `fleeting`,
`literature`, `permanent`; `lang` is `th` or `en`; `hero` is optional. All other keys are rejected.

The supported markdown subset is locked so it round-trips through EmDash's Portable Text without
loss: headings, paragraphs, bold, italic, strike, inline code, links, ordered/unordered lists,
blockquotes, fenced code, images, and `[[wikilinks]]`. Tables, footnotes, MDX, raw HTML, thematic
breaks, setext headings, and indented code are rejected with a file and line number. Nothing is
silently dropped.

## Drafts

Unpublished content is not committed as plaintext. `drafts/<uuid>.enc` is a whole-file
AES-256-GCM envelope:

```json
{"v":1,"alg":"A256GCM","n":"<base64 nonce>","c":"<base64 ciphertext||tag>"}
```

The entry id is the AES-GCM AAD, and the key lives only in the website's Worker secret and the
owner's local environment (`CONTENT_ENC_KEY`). **Losing the key makes the drafts unreadable.** Do not
commit the key here or add it to CI.

To edit a draft outside the admin:

```bash
bun run content:unlock <entry-uuid>   # writes drafts/<uuid>.md + .sha
bun run content:lock   <entry-uuid>   # validates, encrypts, removes the plaintext
```

`lock` refuses to overwrite the remote ciphertext if it changed while unlocked.

## Publish flow

EmDash (the writing surface) saves an entry -> the `content-sync` plugin exports it here -> GitHub
Actions dispatches the site build -> the site is rebuilt and deployed. Published files are plaintext;
pending draft revisions stay encrypted even when the entry itself is published.

## History

The repository was reset to a single initial commit on 2026-11-15. Earlier files were scratch
content and are not part of the site.
