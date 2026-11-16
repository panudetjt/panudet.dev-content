---
title: "A small web is a deliberate web"
slug: "a-small-web"
date: "2026-11-08T06:45:00.000Z"
updated: null
summary: "Fewer dependencies, fewer requests, fewer places for the page to break. A short note on keeping the surface small."
tags:
  - "tools"
  - "web"
para: "projects"
zettel: "fleeting"
lang: "en"
hero: null
---

## Start with the smallest thing that works

The first version of this site has no client router, no analytics script, and no comment widget loaded by default. It has a font, a stylesheet, and the text.

That is not a moral position. It is a way to keep the failure modes visible:

- one font means one place for typography to go wrong;
- inline CSS means no render-blocking request;
- no hydration means the article is finished when the HTML arrives.

## What gets added later

Comments, analytics, and a graph view are allowed — but lazily, after the reading path is complete. The default page should never wait for them.

Small is not the goal. Understanding the whole page is.
