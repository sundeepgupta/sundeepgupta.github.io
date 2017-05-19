---
title: Writing Great Git Commit Messages
tags:
  - craftsmanship
  - source control
id: 86
categories:
  - Uncategorized
date: 2015-01-17 13:34:14
---

People care about _why_ you do things, more than _what_ you do. This notion certainly carries true for our Git commits. Today, I explore a best practice for writing great Git commit messages.

<!--more-->

## The Golden Circle

I recently re-watched one of my favourite TED Talks by Simon Sinek: [How Great Leaders Inspire Action](http://www.ted.com/talks/simon_sinek_how_great_leaders_inspire_action)

In this talk, Simon explains a very simple notion that people are inspired by why leaders do things, not what they do. Well guess what! It's the same for our git commit messages.

![](http://i.imgur.com/QCpZaIk.png)

Simon draws the "golden circle" above (a heck of lot better than mine) and explains that great companies have a profound purpose and are really good at communicating that purpose. With our code, we can think of each commit having its own purpose. And that it's our job to communicate it.

## Why Does This Matter?

You may be wondering, what's the big deal? Why write about something so inconsequential. If you've ever worked on a team, or a long term project, you'll come across code you need to work with and don't understand why it's there. You'll [`git blame`](http://git-scm.com/docs/git-blame) it, find the commit and look at it. After the mini anxiety attack you just encountered hoping that commit wasn't made by you :), you read the commit message hoping to find your answer to "why was this done?".

Further, the vast majority of commit messages I've seen suffer from this problem. Including mine, until not so long ago. In fact, I used to be proud of my commit messages. I thought about them carefully and wrote them as clear as I could.

## The "Aha" Moment

One day, I was pairing with [Craig Savolainen](https://twitter.com/maedhr), a mentor and one of the best developers I know. We were ready to commit our code and I wrote what I thought was a pretty decent commit message, until he explained that my commit message wan't useful. Then told me this:

> Anyone can see WHAT you did just by looking at the code. But the code can never tell someone WHY you did it.

Brilliant! I immediately switched to the new, purposeful commit messages. But it took some time getting used to it, because now I actually had to think - imagine that! Answering the question "Why?" definitely takes more brain power. But now that I've been doing it regularly, it's becoming fluid and natural. A pattern I've noticed is using the word "should" can help frame things.

## Example

Here's an example from an old project. I wrote this commit message in the old _what_ style:

> Card view controller added.

If I were to write this again in the new and improved _why_ style, it would be:

> User should be able to see the card before editing it.

See the difference? The second version provides the context around why this work was done. And this context can mean valuable effort saved by the developer working with this code later. And when that developer has questions, they will come looking for me. So, it also means effort saved by me, especially if I forgot why I originally made this commit, which will likely be the case!

## Conclusion

Writing a Git commit message that explains why we did something as opposed to what we did will help our teams out, and ourselves. A little bit of effort goes a long way here while making another stride in our journey to becoming better software craftsmen. If you have thoughts on this, or more Git message best practices, we'd love to hear about them.

Happy exploring!

* * *

#### Side Note

Obviously, this tip goes beyond Git and applies to any source control tool. I originally was using the term "code commit", but based on feedback, I decided "Git commit" was more clear.