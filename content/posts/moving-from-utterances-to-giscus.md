---
title: "Moving from Utterances to Giscus"
date: 2022-08-30T11:09:21+07:00
comments: true
tags: ["tech"]
draft: false
---

I've used [Utterances](./lets-comment.md) for a while. Using Github's issue feature as a backend for comments is a very elegant solution IM O: no tracking, no ads, simple. But today, I decide to switch to a better alternative - [Giscus](https://github.com/giscus/giscus). Giscus is heavily inspired by Utterances except one thing: instead of using Github issues it uses the fairly new Discussions features to store comments.

So..

## Why migrate?

- **Post reactions**: utterances allows you to add reactions to comments but as an author I’m also interested in the general reception of the post itself. giscus provides this feature.
- **Conversation view**: utterances will simply render comments as a list in the order they have been created. Giscus groups replies to a comment instead. I mean comment is about discussion, right?

## Prepare your site

- Migrate from utterances: [convert the existing issues into discussions](https://docs.github.com/en/discussions/managing-discussions-for-your-community/moderating-discussions#converting-an-issue-to-a-discussion).
- Follow [Giscus](https://giscus.app/), it's quite simple.

`Leave a comment :point_down:`
