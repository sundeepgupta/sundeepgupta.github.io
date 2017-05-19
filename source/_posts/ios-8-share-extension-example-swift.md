---
title: iOS 8 Share Extension Example in Swift
tags:
  - ios
  - swift
id: 32
categories:
  - Uncategorized
date: 2014-06-16 21:41:10
---

While trying to get an iOS 8 Share Extension working in the new Swift language, I ran into some issues due to lack of examples and incomplete documentation. My objective was to let the user share a web page URL with my own custom app. After struggling and digging, here is a working example.

<!--more-->

## Setup

Create a new app extension target by going go File, New, Target in the Xcode menu, then in the left panel select Application Extension and select Share in the right pane. Next, you'll need to name it, and then make sure it belongs to your correct project and embedded in your main target. Remember, app extensions must be distributed with your main app.

Now run the app in Xcode, open a website in Safari and tap the Share icon on the bottom. The Action Sheet should appear and your app should be amongst the sharing options. If you tap on your app, a native share dialog should appear. If you have that working, then we're good to move on.

Xcode creates a target and a folder of files which you'll see in the Project Navigator. If you inspect that folder you should see three files:

1.  ShareViewController.swift
2.  MainInterface.storyboard
3.  Supporting Files/Info.plist

In this example, we'll ignore the MainInterface.storyboard file as we're more interested in simply getting the url from Safari to share.

## Info.plist

First thing's first, lets get our extension configured to share URLs by opening up the Info.plist file and making a few changes. If you right click on it in the Project Navigator, and select Open As, Source Code, we can it in its XML form. We need to change `NSExtensionActivationRule` to a dictionary and add `NSExtensionActivationSupportsWebURLWithMaxCount` as a Number with the value `1`. This lets the OS know that we'll be looking for a web url in our controller. The final result of the `NSExtensionKey` node in the file should look like this:

    &lt;key&gt;NSExtension&lt;/key&gt;
    &lt;dict&gt;
        &lt;key&gt;NSExtensionAttributes&lt;/key&gt;
        &lt;dict&gt;
            &lt;key&gt;NSExtensionActivationRule&lt;/key&gt;
            &lt;dict&gt;
                &lt;key&gt;NSExtensionActivationSupportsWebURLWithMaxCount&lt;/key&gt;
                &lt;integer&gt;1&lt;/integer&gt;
            &lt;/dict&gt;
            &lt;key&gt;NSExtensionPointName&lt;/key&gt;
            &lt;string&gt;com.apple.share-services&lt;/string&gt;
            &lt;key&gt;NSExtensionPointVersion&lt;/key&gt;
            &lt;string&gt;1.0&lt;/string&gt;
        &lt;/dict&gt;
        &lt;key&gt;NSExtensionPointIdentifier&lt;/key&gt;
        &lt;string&gt;com.apple.share-services&lt;/string&gt;
        &lt;key&gt;NSExtensionMainStoryboard&lt;/key&gt;
        &lt;string&gt;MainInterface&lt;/string&gt;
    &lt;/dict&gt;
    `</pre>

    ## Code

    Now that we're configured, lets get into the code and open up ShareViewController.swift. You'll see Xcode stubs out a few methods for us, but to keep this example focused, the only method I'll work in for this example is `didSelectPost`. This is a delegate method that gets called when the user taps the "Post" button in the share dialog. In here, we'll add the code needed to extract the url and user entered text for our share.

    <pre>`override func didSelectPost() {

        //get the itemProvider which wraps the url we need
        var item : NSExtensionItem = self.extensionContext.inputItems[0] as NSExtensionItem
        var itemProvider : NSItemProvider = item.attachments[0] as NSItemProvider

        //pull the URL out
        if (itemProvider.hasItemConformingToTypeIdentifier("public.url")) {        
            itemProvider.loadItemForTypeIdentifier("public.url", options: nil, completionHandler: { (urlItem, error) in
                var urlString = urlItem.absoluteString
                //do what you need to do now, such as send a request to your server with this url
            })
        }

        //default stubbed out code which can pass data back to the host app.
        self.extensionContext.completeRequestReturningItems(nil, completionHandler: nil)
    }

Notice in the completion handler closure, there are no typed arguments. This is because Swift will automatically infer the argument type. If we try to type `urlItem` to NSURL, we'll get compile errors, because that argument is inherently `NSSecureEncoding!`.

And there you have it. Sharing a web url via your own app. I hope this example was helpful for you :)