---
title: "Be Careful With Your HTML element IDs, especially in HTMX"
date: 2024-11-21T09:58:49Z
draft: false
---

I once came across a Reddit thread where people were sharing embarrassing facts about themselves. My favorite was a guy who misheard the saying ["Knowledge is power" - Francis Bacon](https://www.monticello.org/research-education/thomas-jefferson-encyclopedia/knowledge-power-quotation/) as "Knowledge is power, France is bacon". For years, he thought that this was the way the saying went.

## Knowledge is power

It's time for my "France is bacon" moment - I don't know how HTML element IDs work. Do you?

Which one of these is a valid HTML element ID?

- "6d78617a47"
- "55a199a6-888a-499b-9888-b3282646fdb9"
- "my-image 830283" 

![long lost art of meerkat technique in surfing](/meerkat.png)

The image above was meant to do a few things: distract you, raise your awareness of a long lost art of [meerkat technique in surfing](https://www.reddit.com/r/surfing/comments/1gtba5p/whats_the_proper_meerkat_technique) and hopefully obscure explanation below so you don't cheat.

None of those are valid IDs. A valid ID, according to [the only school you need](https://www.w3schools.com/html/html_id.asp):

```...must contain at least one character, cannot start with a number, and must not contain whitespaces (spaces, tabs, etc.).```

That's right, HTML ids cannot start with `numbers` which means things like UUIDs and all sorts of their derivatives are NOT valid HTML element IDs because they MAY start with a digit.

## Why is this a big deal?

It is NOT, probably. Most of the time for most of the people, tbh. Modern browsers don't just shut down and scream at you if HTML element IDs do not conform to the standard. However, once you try something like like this in your javascript code 

`document.querySelectorAll("#6d78617a47")`

this will happen 

`Uncaught SyntaxError: Failed to execute 'querySelectorAll' on 'Document': '#6d78617a47' is not a valid selector.`

And this IS a big deal! Especially if you are doing some [Out of Band 🎺](https://htmx.org/attributes/hx-swap-oob/) (OOB) updates in HTMX which use IDs for targeting. HTMX will receive the payload, try to locate elements using theirs IDs and fail miserably.

This had me scratching my head for a good hour or so. When doing HTMX, my gaze tends to be drawn more and more to the Network tab in devtools and my server console logs rather than javascript console logs. After all, who needs javascript now! (Just kidding, we all need javascript still).

It took me a moment to notice the `document.querySelectorAll` throwing exceptions.

## Post Mortem

Last week I added support for poor man's file uploads to [Tinychat](https://github.com/callmephilip/tinychat). It goes something like this:

- file is picked using standard file component
- upload starts automatically
- upload handler returns some HTML (OOB) representing file in  "uploading" state
- upload is done via Background task
- during upload, each file would get a UUID generated and use to reference this file moving forward for downloads and other stuff (original file name
is kept for decorative purposes only - see [this](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html))
- when upload completes, return updated HTML representing file, this time in "uploaded" state

```html
<a href="/download/3a1b5cfa" id="3a1b5cfa">blah blah</a>
```

> Do NOT do this ⬆️

To my surprise, SOME of my files seemed to be stuck in the endless uploading phase despite uploads completing successfully. I think by now you see where this is going.

HTMX was pushing OOB updates with IDs like `id="3a1b5cfa"` that were rejected by `document.querySelectorAll` while an ID like
`id="a1b5cfa"` would work just fine (because it starts with a letter, not digit). I fixed this by adding a `file-` prefix
to all the element IDs

```html
<a href="/download/3a1b5cfa" id="file-3a1b5cfa">blah blah</a>
```

> This is OK ⬆️

## Before you go

So what did we learn today?

- do not use UUIDs as your HTML element IDs
- keep an eye on JS dev console even when you are on JS fast 
- France is NOT bacon

