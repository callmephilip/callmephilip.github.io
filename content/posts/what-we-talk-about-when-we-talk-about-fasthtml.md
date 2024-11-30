---
title: "What We Talk About When We Talk About FastHTML"
date: 2024-10-01T08:54:34+01:00
draft: true
---

ELI5 (explain like I'm five) should really be ELY5 (explain like you're five). The fact that you have a 5 year old in front of you does not mean you will turn into a clear explainer all of a sudden. Grown-ups sometimes talk down at kids and use big words to hide their lack of understanding of things rather then a means of compression. When you explain things like YOU are five, you put yourself in the mental state of the five year old. You will use simple words, you will tell stories. 

What do we talk about when we talk about FastHTML? Well, like most things it depends. Who am I talking to? Are you a machine learning Python guy who can't bring themselves to learning javascript/typescript and its exotic ecosystem so you can ship things end to end? Are you an indie hacker, a one man band looking for a stack that works and lets you ship fast and with confidence? Are you just dipping your toes into web development and looking for your framework? If this sounds like a setup for a bad joke "An ML guy, an indie hacker and a software intern walk into a bar", it is.

It might be useful to first clarify what FastHTML is NOT. It is NOT a new programming language - you still use Python, javascript, HTML and css. It is not really a framework in the sense of being a monolithic piece of software. It is not an extension of HTML that makes it render faster in your browser. I think the best way to describe FastHTML is as an opinionated collection of very focused open source tools that help you build modern web applications.

"Fast" in FastHTML, stands for 2 things, in my opinion. Fast as in performant at runtime and fast as in "look how fast i built this, mom". FastHTML enables the kind of developer experience that makes you, my dear fellow developer, build and ship useful stuff and do so fast. This is accomplished with sane defaults, almost 0 footprint bootstrap phase - open your editor and hack away. Exhibit A

///////////////////// [insert hello world here] ////////////////////////
///////////////////// [insert quote about Getting started is the product] https://erikbern.com/2024/09/27/its-hard-to-write-code-for-humans

Add to this the fact that FastHTML lets you build and run your web apps inside Jupyter notebooks, and the proverbial feedback loop shrinks into a dot. See how annoying these big words are - FastHTML is none of that. Feel the tension? Good. Keep reading.

Let's talk about something that might have captured your attention from the snippet above. What is this? `Div(Strong("hey"))`. Those old enough to remember their first interaction with React's jsx modules, are probably having a dejavu. Yes dear, it's like there is HTML straight inside your Python module. And yes, you get a hang of it. These are called [FastTags](https://docs.fastht.ml/explains/explaining_xt_components.html). And yes, you can convert your existing HTML to FastTags using this or this or this or this. Now is the good moment and throw your hands in the air, do a ceremonial "this will never work in real world" dance and maybe have a sip of water. We will be here with more details when you are back.

What else is inside FastHTML besides the FastTags that you are already starting to appreciate after that sip of water. FastHTML delegates all the HTTP related chores to Starlette and Uvicorn. Starlette is a framework that simplifies building apps that use ASGI API. Uvicorn is the server connecting your Python code with a browser on the other side of the world. A quick side note regarding FastAPI from the [tech ref](https://about.fastht.ml/tech#sec3).

>Although FastAPI and FastHTML are both built on top of Starlette, and FastHTML is inspired by FastAPI, there are plenty of differences, since they have different purposes. So if you’ve used FastAPI before, don’t assume that everything will be identical!

By default, FastHTML uses SQLite for data access. To make the app dynamic, FastHTML relies on HTMX - AJAX based mechanism to load content from the server and update UI. Unlike more traditional rich web clients that rely on "JSON on wire" types of APIs (REST, GraphQL), HTMX pulls actual HTML markup back from the server. 

Let's take a look

///////////////////////////// [insert some code with HTML] //////////////////////////
/////////////////////////////         SHOW UI              //////////////////////////

Now, if you squint just a tiny bit, this kinda looks like we got ourselves fullstack python going here. In practice you will likely find yourself tweaking some (or a lot) of JS code and maybe even CSS but you can kind see how "Python is all you need" becoming a thing for Web dev with FastHTML.

[///////////////////////// Do a proper walkthrough using movie search example

 == let's inspect a lifecycle of a request with a sample app
 == from browser to server 

 ///////////////////////////]

 Let's talk about 