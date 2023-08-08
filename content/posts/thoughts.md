---
title: 'Thoughts'
date: '2023-03-03T09:33:58+07:00'
tags: ['thoughs', 'non-tech', 'ramblings']
draft: false
comment: true
---

{{< quote info >}}
A collection of interesting thoughts and opinions that I come across
{{< /quote >}}

## Willingness to look stupid

<div align="center">
    <blockquote class="twitter-tweet"><p lang="en" dir="ltr">Willingness to look stupid<a href="https://t.co/0WrVAzLQ6g">https://t.co/0WrVAzLQ6g</a> <a href="https://t.co/6NjysMwbch">pic.twitter.com/6NjysMwbch</a></p>&mdash; Dan Luu (@danluu) <a href="https://twitter.com/danluu/status/1451114505438568448?ref_src=twsrc%5Etfw">October 21, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

I recently read [this article](https://danluu.com/look-stupid/) by Dan Luu (what a clickbait headline). At first, this seems like a terrible (and stupid) idea. Who wants to look stupid? But after read the whole article, I get the point.

People avoid looking stupid even though they may not fully understanding something, and honestly I do the same thing :man_facepalming:. I'd keep my mouth shut rather than ask questions to clarify the problem, most of the time (at work, at some conferences, at university, ...). And I know that I'm not the only one, look smart but being stupid.

There's [a huge difference between stupid and looking stupid. Sometimes looking stupid means you're willing to ask necessary questions to wrap your head around a problem or concept](https://www.linkedin.com/posts/soreniverson_willingness-to-look-stupid-activity-6859517714659123200-cytw/).

_Be willing to look stupid._

Template to look stupid, get it from [Hacker News's thread](https://news.ycombinator.com/item?id=28942189):

```unknow
"I'm going to ask stupid questions now, <insert my question here>"

"Just so I'm understanding the problem, <here's my interpretation of what you just said>."
```

## Leadership

<div align="center">
    <blockquote class="twitter-tweet"><p lang="en" dir="ltr">If you&#39;re a new leader and people all around you walk away, start by looking at you first before labeling everyone a disgruntled employee.<br><br>The more senior and tenured the people who leave, the more it&#39;s time for self reflection.</p>&mdash; Mike Nikles (@mikenikles) <a href="https://twitter.com/mikenikles/status/1562413358787223552?ref_src=twsrc%5Etfw">August 24, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

## Company culture

Basically a common tactic of corporations is to espouse "we are family" but it is just a manipulation tactic for employees to work long-hours and to give their all to the company. The "we are a family" mantra would be fine if it was coupled with a healthy separation of work and life; with the biggest indicator being that healthy working hours (8-9 work hours only) is maintained.

In contrast, something like Netflix culture which is ["we are a team and not a family"](https://jobs.netflix.com/culture) seems like to be more a healthier alternative. On a last point, the "we are family" (i.e. we care about our employees as persons instead of just means or cogs) mantra is fine but it usually turns into "we are a toxic family" (i.e. give your all to the company) instead of it being a "we are a healthy family" (i.e. our company values each employee).

```unknown
We model ourselves on being a professional sports team, not a family.

A family is about unconditional love.

A dream team is about pushing yourself to be the best possible teammate, caring intensely about your team, and knowing that you may not be on the team forever.

Dream teams are about performance, not seniority or tenure.
```

## How to write a resume

Good writing from Huyen Chip: <https://huyenchip.com/2023/01/24/what-we-look-for-in-a-candidate.html>

- **We look for demonstrated expertise, not keywords**
  - Show how you acquired and use that skill in your job.
  - Share your expertise on public channels such as: StackOverflow answers, open source contributions, papers, blog posts
- **We look for people who get things done**
- **We look for unique perspectives**
- **We care about impact, not meaningless metrics**
  - Metrics:
    - How they can be tied to business objectives.
    - Your contribution in achieving that metric.

## Life is too short to work like crazy for most of its part

From [Salvatore Sanfilippo aka antirez aka Redis's author blog](http://invece.org/).

## Success is the result of perfection, hard work, learning from failure, loyalty, and persistence

From [Collin Powell](https://en.wikipedia.org/wiki/Colin_Powell).

## Before you try to do something, make sure you can do nothing

Source: <https://devblogs.microsoft.com/oldnewthing/20230725-00/?p=108482>

When building a new thing, a good thing first step is to build a thing that _does nothing_. That way, you at least know you are starting from a good place. If I'm building a component that performns an action, I'll probably do it in these steps:

0. Write a standalone program to perform the action -> ensure that the action is even possible.
1. Once I have working code to perform the action, I write a component that _doesn't_ perform an action. That at least makes sure I know how to build a component.
2. I register the component for the action, but have the Invoke method merely print the message "Yay!" to the debugger without doing anything else. This makes sure I know how to get the component to run at the proper time.
3. I fill in the Invoke method with enough code to identify what action to perform and which object to perform it on, print that information to the debugger, and return without actually performing the action. This makes sure I can identify which action is supposed to be done.
4. I fill in the rest of the Invoke method to perform the action on the desired object. For this, I can copy/paste the already-debugged code from the step zero.

Too often, I see relatively inexperienced developers dive in and start writing a big complex thing: Then they can’t even get it to compile because it’s so big and complex. They ask for help, saying, “I’m having trouble with this one line of code,” but as you study what they have written, you realize that this one line of code is hardly the problem. The program hasn’t even gotten to the point where it can comprehend the possibility of executing that line of code. I mutter to myself, “How did you let it get this bad?”

**Start with something that does nothing**. Make sure you can do nothing successfully. Only then should you start making changes so it starts doing something. That way, you know that any problems you have are related to your attempts to do something.

## Good code is like a love letter to the next developer who will maintain its

From [Addy's blog post](https://addyosmani.com/blog/good-code/).

## 7 years cycle

{{< figure class="figure" src="/photos/thoughts/we-build-7-years-cycle.png" >}}

## Looking for peaceful

Can't say that I apply that insight consciously but these days there's no over ambitious goal in my head. I just want a peaceful, quiet life. Cooking good food, doing good work, spending time with the people I love. I wanted to be someone famous. Someone that people will look up to and say woah. Now, I'm ok with being a no one.

{{< figure class="figure" src="/photos/thoughts/we-build-another.png" >}}

## Do something, so we can change it

_Just do it_.

From [Allen Pike's blog](https://allenpike.com/2023/do-something-so-we-can-change-it).
