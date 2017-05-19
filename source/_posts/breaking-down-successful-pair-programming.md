---
title: Breaking down successful pair programming
tags:
  - craftsmanship
  - culture
  - pair programming
id: 143
categories:
  - Uncategorized
date: 2015-05-11 23:53:32
---

![Dilbert comic about pair programming](http://i.imgur.com/Q1icg6U.gif)

Explorations into successful pair programming characteristics broken down by common pairing scenarios.

<!--more-->

Pair programming was brought into the mainstream by the Extreme Programming movement and is steadily gaining popularity in production environments. In my two years of pair programming and leading other pairs, I've noticed that sometimes pair programming is fantastic and once, can be even magical. And sometimes, it really sucked. I wanted to explore the reasons behind this and have identified a few patterns that might shed some insight into what it takes for pair programming to work well.

## My pairing experience

I believe its important for you to know what kind of experience I have so you can evaluate this post within better context. I’ve been pair programming almost every day for about two years with two different companies. One was a consulting company where we developed applications for clients. The other was a high growth startup developing a large SaaS product. Through out this time, I paired with over 20 developers all with different backgrounds, personalities, skill, experience and cultures.

**Side note:** Just realizing how much of a benefit _that_ is when you pair program. I've met so many wonderful people and have become friends with many of them.

## Is pair programming worth it?

I wrote a bunch of stuff here on this topic in my draft but deleted it because I don't want to confuse the point of this post. There are many other research papers, opinions, stats, that say one thing or another. I don't think this question matters much in a general context. There are far too many variables.

In the writing below, you will see that I do believe there is good and bad pair programming. So my answer to this question is the most popular answer in our industry - "It depends." :)

And while I may have your attention on this topic, I will mention that I've noticed that pair programming will help the business if it also helps the programmers. (This may deserve a future post.)

## Characteristics of effective pairs

When I think about all of my good and bad pairing experiences, many things that go through my head. Instead of repeating all of that, I summed everything up into this fancy, MBA, consulting style two-by-two matrix.

### Pair programming success matrix

![Pair programming success matrix](http://i.imgur.com/mgDJe9w.png)

### Definitions

#### (Mind the) effectiveness gap

In my observations, the most relevant, distinguishing factor between pairs was the _gap in effectiveness_ between the two individuals. Many things go into how effective a programmer might be - their previous experience, domain knowledge, language knowledge, etc. If we combine all of those things together, we'll get "effectiveness". Perhaps another way to think about "effectiveness" is the potential strength of their output, including speed and quality. For the purposes of this discussion, I'm going to refer to high and low effective programmers as "seniors" and "juniors".

#### Benefit

When pairing, the hope is that the business (and the individuals) will benefit more than if the two worked independently.

### Pairs with small effectiveness gaps

In my experience, these generally worked out better than pairs with large gaps. Perhaps the reason is because the developers have more in common, which equates to instant "free" empathy which helps improve their interactions. I believe the points outlined in the matrix are clear. Essentially, strong pairs with small effectiveness gaps have a good chemistry. They understand and respect each other, communicate often, and help each other learn. It results in a very satisfying and mutually beneficial interaction. It's usually a lot of fun too.

If we dig a level deeper, there are different characteristics between senior and junior pairs.

#### Junior pairs

It is my observation that junior pairs provide more benefit than senior pairs. Why? Probably because junior developers have weak opinions, their egos are smaller, and they care more about learning as opposed to being "right".

I also believe there's the "new kids in school" effect happening. Imagine you're a kid and your family moves to a new town in the middle of the school year. If there was another kid in your class who also joined around the same time as you, you'd probably become friends rather easily since you're both in the same situation. It's the same with pairing. If you're both new to the app/technology/language/whatever, you'll instantly have more empathy for one another. You'll want to explore similar things, get excited about solving the same type of problems and generally collaborate well.

When two juniors pair, the primary goal should be to **learn**.

#### Senior pairs

When both developers are seniors, it's harder to get a good match for the same reasons. Every programmer is different, and has different opinions. The more experience you have, the stronger those opinions are. Also, when you know a lot, it is possible for it inflate your ego. When one person has a large ego, the pair may yield poor decisions due to being insecure or the there person "tiptoeing" around to ensure they don't dent their pair's ego.

Even worse is when both pairs have large egos. In this case they argue (not debate) a lot and it can become tense with animosity. The worse I've seen is when the navigator would physically walk away from the workstation for extended periods of time. The pair wasn't even communicating at this point and obviously in negative benefit territory.

Good senior pairs have a lot of respect for each other. They talk a lot. They debate, discuss and plan. They work at a professional level, not personal. It is a thing of beauty. And the business benefits greatly with strong output.

When two seniors pair, the primary goal should be to **produce**.

### Pairing juniors with seniors

Putting together a senior and junior requires a different approach. This pairing arrangement tends to happen when new developers join the company or switch into a new development context. The new, "junior" might be paired with a "senior" with the expectation for the junior to ramp up quickly. If the senior is not the right person, or has the wrong expectations, this will turn out badly. and risk hurting the junior's morale.

#### Expectation

Leadership must understand this arrangement is about ramping up the junior as fast as possible. But that must be understood by everyone who has a stake in this pair - the other developers, designers, QA, etc., and, especially the senior. The pair should not have any pressure to deliver, because once they do, it will ruin the pairing dynamic. It will turn into frustration on both sides because the senior will be slowed down by, or will run ahead of the junior.

> Output will be slow while mentoring takes place.

#### Mentoring

Once the senior understands that their role is to mentor, and not to pump out stories, only then can the pair start to engage in beneficial work. But there's more to consider.

> Good mentors listen, answer questions, observe and ask _leading_ questions.

This is really hard to do because as human beings, we tend to do the opposite. So here's a reminder:

Do NOT tell them what to type, or tell them what to do or take over and show them. Stop yourself. Let them make the mistake you see coming. Let them feel some of the consequences. Let them think about why that was bad. Let them correct it. Let them ask you questions along the way. Try to answer with _questions_. If they get stuck, lead them out with _questions_.

I remember I was really under the gun to get stuff done while I was pairing with a brand new co-op student. I knowingly steamrolled my pair and continuously apologizing, which probably didn't make him feel any better. Looking back, I should have changed the pairing arrangement or found a way to take the pressure off.

It takes a ton of patience, self-confidence and empathy to be a good mentor. We want build the junior up, not show them what you know. From what I've seen, good programming mentors are lacking in our industry. Everyone is too busy trying to get stuff done or they just don't have the human skills to do it well. Some questions that may help determine if someone is a good mentor are:

Do they share knowledge and help juniors often? Do they get defensive about their skills or opinions? Are they comfortable with sacrificing for the good of the team?

When pairing a senior and junior together, the primary goal should be **mentoring and learning**.

## Leadership's role

### Culture

In any situation, good things happen when there is understanding between people. Pair programming is no different. Understanding comes from empathy, communication and self-confidence. Business and technical leaders can set the example so it can propagate down through the rest of the team. Culture comes from the top. And it's not what they say it is, it's how they _act_ that defines culture.

### Expectations

As I've identified above, each pairing scenario is different. It's important to recognize them as different, understand what will make it beneficial, and communicate that to everyone who needs to know so that expectations are clear and in alignment with the pairing dynamic.

## Conclusion

This particular subject reinforced for me that programming is as much about people as it is about technology. We are people, writing software for people, in collaboration with other people. The technical skills are obviously required, but being a great programmer goes hand in hand with being a great person.

Happy exploring!