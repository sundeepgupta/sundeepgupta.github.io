---
title: iOS Testing Tips with Kiwi
tags:
  - ios
  - testing
id: 30
categories:
  - Uncategorized
date: 2014-01-22 21:36:44
---

For those getting to know [Kiwi](https://github.com/allending/Kiwi), as an iOS testing framework, here are a few tips.

<!--more-->

This post assumes you're already familiar with the basics of Kiwi testing. For more information about Kiwi, you can look at the [Kiwi documentation](https://github.com/allending/Kiwi/wiki) or read [Test Driving iOS Development with Kiwi](http://editorscut.com/Books/001kiwi/001kiwi-details.html).

## Private Methods/Properties

Before learning about this, we were exposing private methods and properties in order to test them. And felt dirty in the process. To avoid this, use is a class extension. In your test file, create an interface to the class that has the private stuff you need to test. Then simply declare any private methods/properties and voila!

    #import "Kiwi.h"
    #import "SGStopWatch.h"

    @interface SGStopWatch (StopWatchTesting)
    @property (nonatomic, strong) UILabel *clockLabel;
    - (void)resetClockLabel;
    @end

    SPEC_BEGIN(StopWatchSpec)
    `</pre>

    ## Block Variables

    As everything in Kiwi is done via blocks, if you like to reset the state by instantiating (or replacing) your variable inside of a block, you need to preface your variable declaration with `__block`. This is especially useful for any `beforeEach` blocks you create.

    <pre>`describe(@"Stopwatch", ^{
        context(@"when stopped", ^{
                    __block SGStopWatch *stopWatch;
                    beforeEach(^{
                            stopWatch = [SGStopWatch new];
                    });
    `</pre>

    ## Shortcut Tests

    A quick and dirty tip, but worth mentioning. You can test for `nil` using shorthand like this: `[stopWatch shouldBeNil]` or `[stopWatch shouldNotBeNil]`

    ## Pending Tests

    If you like stubbing out tests, and then coming back to them later, you can mark your tests "pending" by starting them with `pending` instead of `it`. Pending tests are not actually tested, so it won't affect your overall test result. What I like about using `pending` is that it produces a yellow compiler warning so you won't forget about it.

    <pre>`pending(@"should not be nil", ^{
            //I\'ll come back to this
    });
    