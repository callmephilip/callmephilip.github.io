---
title: "Exploring Codebases With Gemini and Mandark"
date: 2025-02-18T19:07:02Z
draft: false
---

Taking mandark + gemini 2.0 flash for a spin. Let's look at [https://github.com/mary-ext/skeetdeck](https://github.com/mary-ext/skeetdeck) to figure out how it's dealing with rich text facets and embeds.

Git cloned skeetdeck  with `git clone https://github.com/mary-ext/skeetdeck`. Ask mandark copy all code from app/api:

`npx mandark app/api -a -c`

This goes straight into the clipboard and looks smth like this:

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreie6ljzb2dnt6gcqvrtyqiaatt3u6vl2kru72idjgvigamu6w7roua@jpeg)


> [Philip Nuzhnyi - @callmephilip.com](https://bsky.app/profile/callmephilip.com) **2025-02-18 13:06**

Loading the entire thing into gemini. Looks pretty wild but Gemini does not seem to care:

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreibmlkie7cpoe4dtf7qbooxmtpts5grdr5zfbqfmu5tmolxz7esdja@jpeg)


First run, let's try to locate facet extraction flow. My question is: 

`can you identify which part of it is dealing with text parsing to extract what is called "facets" - these should be links, metions and tags.`

Gemini's response below. Saving to GH for posterity - [github.com/callmephilip...](https://github.com/callmephilip/tinychat-at-proto/issues/3#issuecomment-2665594097):

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreiggucduvk5dvlnplrlzsfvbkml5szwrdzsl3kcaps6nzcixytbv7u@jpeg)


> [Philip Nuzhnyi - @callmephilip.com](https://bsky.app/profile/callmephilip.com) **2025-02-18 13:06**

Next, let's figure out how embeds are processed. My query: 

`how are embeds processed? these typically include external sites or posts. This thing is doing all the work and flattering me at the same time`

To which, we get:

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreic43b4g2t4sauw5g4ns4aqr34kqkmc2sd4l3kzciofs4hxeavej7e@jpeg)

Spot checking this - looks pretty reasonable. I like that it also picked up on get-link-meta API where metadata gets pulled for external links. Saving for later [github.com/callmephilip...](https://github.com/callmephilip/tinychat-at-proto/issues/3#issuecomment-2665626223). 

Interesting that it incorrectly points to `app/api/util/post.ts` instead of `app/api/utils/post.ts`

Let's try something else. I will need this stuff running on top of Deno with Hono:

`can you convert app/api/queries/get-link-meta.ts into a micorservice using Deno and Hono please`

> Please is always a nice touch - these machines don't forget, always be nice to them

And it returns a bunch of code, see [here](https://github.com/callmephilip/tinychat-at-proto/issues/3#issuecomment-2665645841)

... which works!

`curl "http://localhost:8000/getLinkMeta?url=https://bsky.app/profile/fchollet.bsky.social/[...]"`

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreid3s6cwqklt33rrzrn3kzxnrzxxgspxihhdyuh6gnz25udyfuvmma@jpeg)

---

I've asked Gemini to impersonate Sterling Archer and it's amazing at it!  "Let's get this done, I already asked Woodhouse to prepare a Negroni and my ice isn't going to stay frozen FOREVER"

![no image description](https://cdn.bsky.app/img/feed_fullsize/plain/did:plc:ubdeopbbkbgedccgbum7dhsh/bafkreidv34jfftjval7uc4w7cu65vaywuirzkzlm34yq25d2wbakgcd5qi@jpeg)


Getting strong jailbroken "Ashley too" vibes [www.youtube.com/watch?v=-qIl...](https://www.youtube.com/watch?v=-qIlCo9yqpY)

---

> This post was generated using [bsky2md](https://bsky2md.deno.dev/?url=https://bsky.app/profile/callmephilip.com/post/3lihdbqbfu22p)