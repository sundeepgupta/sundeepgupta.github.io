---
title: iOS Modal Call Sequence
tags:
  - ios
id: 35
categories:
  - Uncategorized
date: 2014-12-10 21:45:39
---

I needed to call these methods manually for a custom modal transition with help of [this example code](https://github.com/soleares/SOLPresentingFun). Instead of relying on my memory of the sequence of calls view lifecycle delegate calls, I document them here. And it's nice to know they are consistent for both iOS 8 and 7.

<!--more-->

## Presenting the Modal

1.  **Modal** viewDidLoad
2.  **Presenter** viewWillDisappear:animated:
3.  **Modal** viewWillAppear:animated:
4.  **Modal** viewDidAppear:animated:
5.  **Presenter** viewDidDisappear:animated:

## Dismissing the Modal

1.  **Modal** viewWillDisappear:animated:
2.  **Presenter** viewWillAppear:animated:
3.  **Presenter** viewDidAppear:animated:
4.  **Modal** viewDidDisappear:animated: