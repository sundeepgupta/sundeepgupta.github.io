---
title: Dependency injection example in Swift
tags:
  - coding
  - design patterns
  - swift
id: 128
categories:
  - Uncategorized
date: 2015-03-10 10:14:59
---

The term "dependency injection" is a popular, but also confusing term, especially for newer developers. This post attempts to demystify things with a simple example done in Swift.

<!--more-->

## What is dependency injection?

### Complicated explanation

[Wikipedia](http://en.wikipedia.org/wiki/Dependency_injection) provides a definition with perhaps more fancy terms along with a very general example, which I don't find very helpful.

### Over simplistic explanation

[This StackOverflow post](http://stackoverflow.com/questions/130794/what-is-dependency-injection) has several answers, including one that quotes James Shore:

> Dependency Injection" is a 25-dollar term for a 5-cent concept. [...] Dependency injection means giving an object its instance variables. […].

Although in his blog post he goes into more detail, this statement is misleading. Its more complicated than just providing instance variables.

### My explanation

Having a good understanding of what problem dependency injection can solve might be a good place to start. If you can think of a time where you have existing, working code and then you're given the task of adding behaviour you didn't originally anticipate. Dependency injection can help with this sort of thing, so that the new behaviour is easy to add.

Here's my attempt at explaining it:

> Dependency injection is recognizing the various changes your object needs to make. Instead of having your object make those changes, you move the changes out into their own "changer" objects (dependencies), and then pass (inject) them into your object for it to use. What gives dependency injection its power and usefullness is creating an abstraction from these dependencies to provide a unified interface. This way you your object has to care about this abstracted interface and not the actual dependency it recieves (which helps acheive loose coupling).

Examples are usually more helpful, so lets get to it.

## Dependency injection example in Swift

This example is inspired by an example Sandi Metz did in a workshop I attended which helped clarify things for me. The following example shows the evolution of the `Song` class which recites the cumulative song, [The House That Jack Built](https://www.youtube.com/watch?v=7sDSYVfnj_E). For conciseness, I only use the first three lines of the song:

> This is the house that Jack built.
> 
>   This is the malt that lay in the house that Jack built.
> 
>   This is the rat that ate the malt that lay in the house that Jack built.

We'll start with a single requirement, then add requirements that help reveal the need for dependency injection. I've created playground files located in [this Github repository](https://github.com/sundeepgupta/dependency-injection-example) or you can [directly download them](https://github.com/sundeepgupta/dependency-injection-example/archive/master.zip). At the bottom of each playground file, there are "tests" to make sure our code is working.

### Start - normal case

The initial requirements are to recite the song, or a specified line. This is in the file `0-Plain.playground`.

    class Song {
        private let lineStart = "This is "
        private let lineEnd = "."
        private let phrases = [
            "the house that Jack built",
            "the malt that lay in",
            "the rat that ate"
        ]

        func line(number: Int) -&gt; String {
            let linePhrases = self.phrases[0...number-1].reverse()
            let line = " ".join(linePhrases)
            return self.lineStart + line + self.lineEnd
        }

        func recite() -&gt; String {
            var lines = [String]()
            for i in 1...self.phrases.count {
                lines.append(self.line(i))
            }
            return "\n".join(lines)
        }
    }
    `</pre>

    ### Add first requirement - repeat each phrase

    This requirement means we want each phrase in each line to repeat. For example, the second line would be:

    > This is the malt that lay in the malt that lay in the house that Jack built the house that Jack built.

    There are several options to add this requirement and it doesn’t matter which one we choose as they'll converge to the same fundamental issue. I choose to do the following which can be seen in `1-Repeat.playground`.

*   Make the `phrases` property mutable.*   Create a custom `init` that takes a `repeat` boolean.
*   Conditionally modify `phrases` based on `repeat`.
    <pre>`class Song {
    // ...
        private var phrases = [
            "the house that Jack built",
            "the malt that lay in",
            "the rat that ate"
        ]
        init(repeat: Bool) {
            if repeat {
                self.phrases = self.phrases.map { phrase -&gt; String in
                    return phrase + " " + phrase
                }
            }
        }
    // ...    }
    `</pre>

    Here, we have a new dependency that the `phrases` array needs to be changed to accomodate the new requirement. Notice `Song` handles the dependency internally, it's not being injected in.

    ### Add second requirement - reverse the song

    Now we want the ability to recite the song and each lines backwards. For example, line two would be:

    > This is the malt that lay in the rat that ate.

    To add this, I simply follow the pattern already established. The changes made in `2-Reverse.playground` are:

*   Instead of a boolean now, pass in a `songType` to indicate how the song should be sung.
*   Add the code to reverse `phrases`.
    <pre>`class Song {
    // ...
        init(songType: String) {
            if songType == "repeat" {
                self.phrases = self.phrases.map { phrase -&gt; String in
                    return phrase + " " + phrase
                }
            } else if songType == "reverse" {
                self.phrases = self.phrases.reverse()
            }
        }
    // ...
    }
    `</pre>

    Now we have two dependencies that `Song` internally handles. This code is a bit wierd as we still have to pass in a string for the normal case. We could solve this in a few ways, but that's not the focus so lets leave it for now.

    What if we are to add a third requirement now? You're probably feeling the urge to look for a better way than to just add a third `songType`. The code we have now isn't ["open" for change](http://en.wikipedia.org/wiki/Open/closed_principle), in that, to add new new behaviour, we need to _modify_ our code. We want to be able to only _add_ new code.

    ### Move dependencies out

    First, lets move the dependent code their own classes and modify our `init` accordingly to use them in `3-Classes.playground`:

    <pre>`class Repeater {
        func result(list: Array&lt;String&gt;) -&gt; Array&lt;String&gt; {
            return list.map { item -&gt; String in
                return item + " " + item
            }
        }
    }

    class Reverser {
        func result(list: Array&lt;String&gt;) -&gt; Array&lt;String&gt; {
            return list.reverse()
        }
    }

    class Song {
    // ...
        init(songType: String) {
            if songType == "repeat" {
                self.phrases = Repeater().result(self.phrases)
            } else if songType == "reverse" {
                self.phrases = Reverser().result(self.phrases)
            }
        }
    // ...
    }
    `</pre>

    Now that our dependencies live outside of `Song`, the code is a little bit nicer, but it's still not open for change. We still have the conditional in our `init` and our `Song` still must know about both `Repeater` and `Reverser` dependencies. To solve this problem, we need the ability to inject these dependencies into `Song`. But how? By creating an abstraction of and so we can provide a unified interface to them. There are three ways to do this in Swift: inheritance, protocols, and closures. For this example, I choose to use a protocol.

    **Note:** If we were doing this in a dynamically typed language like Ruby, we wouldn't need this step since we've already given our dependencies the same interface, `result()`.

    ### Inject dependencies

    We create our protocol `Remixer` and have our dependencies conform to it. Now we can inject any class conforming to `Remixer` into `Song`'s `init` as seen in `4-Injectction.playground`.

    <pre>`protocol Remixer {
        func result(list: Array&lt;String&gt;) -&gt; Array&lt;String&gt;
    }

    class Repeater : Remixer { // ... }

    class Reverser : Remixer { // ... }

    class Song {
    // ...
        init(remixer: Remixer?) {
            if remixer != nil {
                self.phrases = remixer!.result(self.phrases)
            }
        }
    // ...
    }
    `</pre>

    That's it! That is dependency injection.

    You might be wondering about why `init` takes an optional `Remixer` value. That was a choice I made. We could have also created another `Remixer` called `Keeper` that does returns the list without modification, and inject a `Keeper` into `init`. Or also set a `Keeper` as the default paramater value.

    ### Add third requirement - randomize the song

    This requirement means to play the song in a random order. And now that `Song` is open to change via dependency injection, this requirement is trivial to add. We simply add a new `Remixer` called `Shuffler`and inject it in as shown in `5-Shuffle.playground`.

    <pre>`import Foundation

    class Shuffler : Remixer {
        func result(list: Array&lt;String&gt;) -&gt; Array&lt;String&gt; {
            var newList = list
            let listCount = count(newList)
            for i in 0..&lt;(listCount - 1) {
                let j = Int(arc4random_uniform(UInt32(listCount - i))) + i
                swap(&amp;newList[i], &amp;newList[j])
            }
            return newList
        }
    }

## Conclusion

Doing examples like these help me understand things better and hope I've helped you understand dependency injection and how it can be useful.

Happy exploring!