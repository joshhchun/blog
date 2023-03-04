---
title: Pym - Code/Image Pastebin & URL Shortener
summary: "so i made website"
date: 2022-07-02T16:16:02-04:00
url: "/projects/pym"
weight: 1
showtoc: true
tags: ["golang", "react", "postgresql", "vite", "typescript"]
draft: false
cover:
  image: "/posts/projects/pym/pictures/header.png"
  alt: "header picture"
  caption: ""
---

### Tech Stack

_Golang, Typescript + React.JS, PostgreSQL, Vite, Docker, Nginx, & Caddy_

## What is it?

Pym is a syntax-highlighted code/image pastebin and URL shortener that allows you to share your code/images easily. You are able to paste the code or upload the file itself. The name originates from Marvel's _pym particles_, which are the particles that allow Ant-Man and Wasp-Woman to shrink.

The idea is that you are able to shrink your entire code file or image into a tiny unique URL to share with whomever. Nerdy? Yes. Cool? A little. But as the saying goes,

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> &mdash; Phil Karlton

Once you save/upload a post, a unique short URL with a 4 character ID will be generated ([https://pym.jchun.me/yzYj](https://pym.jchun.me/yzYj)) for you to share.

Every post is created with a default life time of **15 days**. However, I wanted to implement a LRU-like deletion mechanism, so a post's life time is reset to 15 days every time it is visited to prevent a relevant post from being deleted.

![img](/posts/projects/pym/pictures/TUT2.gif)
|:--:|
| _Creating a new post with a code snippet and sharing URL_ |

## Why?

The reason for Pym's creation can be boiled down to a couple of points.

- I wanted an easy & pretty way for my friends and I to share code
- I often use [IRC](https://en.wikipedia.org/wiki/Internet_Relay_Chat) and it can't handle large images, messages, or links very well
- Being more experienced with Python Web Dev, I wanted to explore the Javascript environment (I love it)

There have been so many occasions where my friends and I wanted to share some code with each other, and there just _simply was not a good means to do so_.

We have tried everything from the OG [Pastebin](https://pastebin.com/), Google Docs, to Email (yes.. _email_), but there was no solution that was _quite_ right. The lack of syntax highlighting or terrible/messy UI... it seemed like there had to be a better solution - _and that is why I made Pym._

## How to use?

My main goal when developing this application was to _keep it simple_.

Yes, features are great but having too much configuration can very easily lead to a messy UI, and that is what turned me away from a lot of alternative websites.

To make a new post, a user just has to click on the `create` button on the top right corner of the home page (<https://pym.jchun.me>), and from there they can choose to either

### Paste text

![newtext](/posts/projects/pym/pictures/newtext.png)

- Simply paste some code in the textbox and choose the programming langauge (used for accurate syntax highlighting and supports more than 135 languages)
- Upon save, user will be redirected to the new unique URL for sharing.
- From the CLI you can also make a cURL request:

```bash
$ curl -d '{"group": "text", "language": "plaintext", "value":"$TEXT"}' \
 -H 'Content-Type: application/json' https://pym.jchun.me/api/save
```

`Response > {"shortId":"6t5a"}`

### Upload a file

![newfile](/posts/projects/pym/pictures/newfile.png)

- User can either drag and drop a file or browse.
- If the file is of a popular programming language, it will automatically be converted to a "paste" post with syntax highlighting.
- Upon upload, user will be redirected to the new unique URL for sharing.
- From the CLI you can also make a cURL request:

```bash
$ curl -F "files=@$FILE" https://pym.jchun.me/api/save
```

`Response > {"shortId": "324f"} `

### Shorten a URL

![newurl](/posts/projects/pym/pictures/newurl.png)

- Simply paste the long (or average sized?) URL into the box and click the upload icon on the right.
- The newly generated short URL will be displayed in the box.
- From the CLI you can also make a cURL request:

```bash
$ curl -d '{"group": "link", "value":"$LINK"}' -H 'Content-Type: application/json' \
https://pym.jchun.me/api/save
```

`Response > {"shortId":"6t5a"}`
