---
title: "Why this site is static first"
slug: "why-static-first"
date: "2026-11-14T09:30:00.000Z"
updated: "2026-11-15T10:00:00.000Z"
summary: "A personal site is read far more often than it is written. Prerendering every public page keeps the reading path cheap and the writing path separate."
tags:
  - "architecture"
  - "performance"
  - "web"
para: "projects"
zettel: "literature"
lang: "en"
hero: null
---

## The writing surface is not the reading surface

The admin needs a database, authentication, and a runtime. The public pages need none of those. Mixing the two makes every reader pay for the editor.

This site keeps them apart:

- EmDash owns writing and publishing.
- The content repository owns the markdown.
- Astro prerenders every public route at build time.
- Cloudflare serves static assets; the Worker only handles `/_emdash/*` and `/preview/*`.

That also means the site can be rebuilt from a cold clone without a database. The build reads files, not rows.

## What the budget buys

A fast page is mostly a sequence of deletions. No blocking scripts. One font request. No client-side router. No hydration on an article. The remaining cost is HTML, CSS, and the text itself.

```ts
const budget = {
  html: "16 KB gzip",
  requests: 8,
  blockingJs: 0,
  font: "32 KB",
};
```

```html live
<div id="inspector">
  <button id="inspect-btn" type="button">Inspect budget</button>
  <span id="inspect-out">Zero blocking scripts</span>
</div>
<script>
  console.log("Budget inspector ready");
  document.getElementById("inspect-btn").onclick = () => {
    console.log("Inspect clicked");
    document.getElementById("inspect-out").textContent = "All budgets passing!";
  };
</script>
```

> Speed is a feature only when it is measured. Otherwise it is a hope.

## The tradeoff

Publishing is not instant. A save becomes a commit, the commit triggers a build, and the build takes a minute or two. For a personal knowledge base, that delay is fine — and it keeps the public surface small enough to understand.

See [[ทำไมต้องเขียน-second-brain]] for the other half of the argument: why the notes are public at all.
