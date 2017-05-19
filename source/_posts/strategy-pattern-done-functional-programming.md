---
title: Strategy pattern done with Functional Programming
tags:
  - coding
  - design patterns
  - functional programming
  - swift
id: 184
categories:
  - Uncategorized
date: 2016-01-17 21:51:00
---

A quick, simple technique to help transition your Object Oriented code to a more functional style.

<!--more-->

## Intro

Showers are a mysterious place where for some reason, insights appear. I made a mini break through with the practice of Functional Programming (FP) in the shower. In hindsight, it's not groundbreaking, but I think it's worth discussing, especially if you're struggling to grasp how FP can be used in practice.

A while back, a functional approach nicely solved a problem I was having, where I was going to implement some type of interface with a set of concrete classes, but then realized a function would be easier and cleaner. So, I'm calling this out as a technique.

> FP can be used in place of the common Object Oriented practice of [programming to interfaces, not implementations.](http://stackoverflow.com/questions/383947/what-does-it-mean-to-program-to-an-interface)

I don't know yet if this is true in general, but it's certainly true for the Strategy Pattern. I'll use my previous post, [Dependency injection example in Swift](http://sundeepgupta.ca/dependency-injection-example-in-swift/) to illustrate this.

## Code

The Swift Playground for this post can be found in [this repo](https://github.com/sundeepgupta/functional-strategy-pattern) if you'd like to download it.

## Object Oriented example

First, let's review the OO example. This was inspired by a Sandi Metz Ruby exercise I experienced a while back, and ported to Swift to illustrate what exactly Dependency Injection is and how it can be useful. The technique described in the post is more precisely known as the [Strategy Pattern](https://en.wikipedia.org/wiki/Strategy_pattern).

Quickly, the `Song` class has a `recite()` method to sing a short song consisting of an array of phrases. Using the Strategy Pattern, we can instantiate and recite the song, but remixed in a certain way, at runtime. To do this, we to pass in an object that conforms to the `Remixer` protocol (in other OO languages, like Java, a protocol is called an interface). In the example, we implement `Repeater` and `Reverser` to recite the song with repeated phrases and reversed phrases, respectively.

    class Song {
        private let lineStart = "This is "
        private let lineEnd = "."
        private var phrases = [
            "the house that Jack built",
            "the malt that lay in",
            "the rat that ate"
        ]

        init(remixer: Remixer?) {
            if remixer != nil {
                self.phrases = remixer!.result(self.phrases)
            }
        }

        func line(number: Int) -&gt; String {
            let linePhrases = self.phrases[0...number-1].reverse()
            let line = linePhrases.joinWithSeparator(" ")
            return self.lineStart + line + self.lineEnd
        }

        func recite() -&gt; String {
            var lines = [String]()
            for i in 1...self.phrases.count {
                lines.append(self.line(i))
            }
            return lines.joinWithSeparator("\n")
        }
    }

    protocol Remixer {
        func result(list: [String]) -&gt; [String]
    }

    class Repeater: Remixer {
        func result(list: [String]) -&gt; [String] {
            return list.map {
                return $0 + " " + $0
            }
        }
    }

    class Reverser: Remixer {
        func result(list: [String]) -&gt; [String] {
            return list.reverse()
        }
    }

    Song(remixer: nil).recite()
    Song(remixer: Repeater()).recite()
    Song(remixer: Reverser()).recite()
    `</pre>

    ## Functional example

    In the OO example, above, `Song` uses the `Remixer` protocol defined method `result()` on the object being passed in. Now, instead of passing in an object, which conforms to the `Remixer` protocol, which implements `result()`, in a sense, we'll pass in the `result()` _function_ directly. Easy, right? Let's look at the code.

    <pre>`class Song {
        private let lineStart = "This is "
        private let lineEnd = "."
        private var phrases = [
            "the house that Jack built",
            "the malt that lay in",
            "the rat that ate"
        ]

        init(remixer: Remixer?) {
            guard let r = remixer else { return }

            self.phrases = r(phrases)
        }

        func line(number: Int) -&gt; String {
            let linePhrases = self.phrases[0...number-1].reverse()
            let line = linePhrases.joinWithSeparator(" ")
            return self.lineStart + line + self.lineEnd
        }

        func recite() -&gt; String {
            var lines = [String]()
            for i in 1...self.phrases.count {
                lines.append(self.line(i))
            }
            return lines.joinWithSeparator("\n")
        }
    }

    typealias Remixer = [String] -&gt; [String]

    let repeater: Remixer = { lines in
        return lines.map {
            return $0 + " " + $0
        }
    }

    let reverser: Remixer = { lines in
        return lines.reverse()
    }

    Song(remixer: nil).recite()
    Song(remixer: repeater).recite()
    Song(remixer: reverser).recite()

`Song` looks identical to the OO version except for inside `init()`. `init()` still takes a parameter of type `Remixer?`, but now, `Remixer`'s type is a function, not a protocol. In place of our concrete classes which implemented the dynamic behaviour in `result()`, we define constants of type `Remixer`, our function type.

If you have dynamic behaviour in your OO code and you're looking to introduce FP into your practice, this is one approach that can help you get into the swing of it.

Happy exploring!