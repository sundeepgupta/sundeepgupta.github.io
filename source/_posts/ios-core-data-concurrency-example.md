---
title: iOS Core Data Concurrency Example
tags:
  - ios
id: 27
categories:
  - Uncategorized
date: 2013-07-07 21:29:25
---

An iOS Core Data concurrency example showing how one data store can be hot swapped with another while performing read and write operations.

<!--more-->

I'm working on a project that requires downloading a JSON data feed from a web server to update local data on an iOS device using a Core Data persistent store.

## Problem

1.  The data is not indexed by anything. Thus, I could not _update_ the local objects directly. I had to do a full data set replace, which meant deleting all the existing objects and create new objects on each sync.
2.  The data set was over 60,000 records and over 5MB. Saving to the local store via Core Data operations was locking up the UI.

3.  The UI needed access to the old data while the new data was being downloaded and written to disk.

## Solution

I was aware of asynchronous `NSURLConnection` techniques for the download to free up the UI thread, but that wouldn't help with the issue of UI locking when saving data to the local store. So, I dug deeper into Core Data and learned about its ability to handle concurrency, which was improved for iOS 5+.

To summarize, the solution I went with was to create a new, temporary, background `NSManagedObjectContext` (MOC) for the new data. Once all the objects were written to the MOC, I "flipped the switch" by replacing the existing `NSPersistentStore` then saved the MOC. This allowed all the data downloading and saving to be done in the background and being able to do a full replace update without the UI ever knowing.

I chose to do it this way because MOCs are cheap, and it is simpler to implement than other setups like nested MOCs. You can read more about that on [Florian Kugler's blog](http://floriankugler.com/blog/2013/4/2/the-concurrent-core-data-stack).

## Implementation

I won't get into the asynchronous downloading because there are several resources online that teach it:

*   [Apple Documentation - Using NSURLConnection](http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836-BAJEAIEE)
*   [Code With Chris - NSURLConnection By Example](http://codewithchris.com/tutorial-how-to-use-ios-nsurlconnection-by-example/)

What I want to show is the actual Core Data code as I didn't find any comprehensive examples online and Apple's documentation on this topic is lacking.

When I started the project, I checked off the Core Data option, which sets up the Core Data stack inside of your **AppDelegate** class.

![](http://content.screencast.com/users/SundeepG/folders/Do%20not%20delete/media/00a4880a-0c07-4435-bc95-78c84afb038b/00000336.png)

After setting up my asynchronous `NSURLConnection`, in the delegate method, `connectionDidFinishLoading:connection`, this is where I was saving the data:

    - (void)connectionDidFinishLoading:(NSURLConnection *)connection {
        //prepare the data into an array of objects
        NSData *data = [self dataForConnection:connection];
        NSArray *objects = [self parseData:data];

        //setup tempMoc
        UIApplication *app = [UIApplication sharedApplication];
        AppDelegate *appDelegate = (AppDelegate *)app.delegate;
        self.moc = appDelegate.managedObjectContext;
        NSPersistentStoreCoordinator *storeCoordinator = self.moc.persistentStoreCoordinator;
        NSManagedObjectContext *tempMoc = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];
        tempMoc.persistentStoreCoordinator = storeCoordinator;

        NSString *entityName = NSStringFromClass([Compatibility class]);

        //specify the thread for the callback after all the saving is finished
        __block dispatch_queue_t currentQ =  dispatch_get_current_queue();

        //save and do anything else for this particular MOC on a background thread inside the block
        [self.tempMoc performBlock:^{        
            for (NSDictionary *newObjectDict in objects) {
                Compatibility *object = [NSEntityDescription insertNewObjectForEntityForName:entityName inManagedObjectContext:self.tempMoc];
                object.prod1 = newObjectDict[@"prod1"];
                object.prod2 = newObjectDict[@"prod2"];
                object.result = newObjectDict[@"result"];
            }

            //after all the stuff in the block is done, execute this block on the thread specified
            dispatch_sync(currentQ, ^(){
                [tempMoc save:nil];

                //flip the switch
                [self.moc lock];
                [self.moc reset];
                NSPersistentStore *store = storeCoordinator.persistentStores.lastObject;
                NSURL *storeUrl = store.URL;
                [storeCoordinator removePersistentStore:store error:nil];
                [[NSFileManager defaultManager] removeItemAtURL:store.URL error:nil];
                [storeCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeUrl options:nil error:nil];
                [self.moc unlock];
                self.moc = tempMoc;
                tempMoc = nil;
            });
        }];
    }

Let's go over the important parts here.

When setting up your temporary MOC, to make sure to initialize it with `initWithConcurrencyType:NSPrivateQueueConcurrencyType` so it can open and manage its own background thread and queueing operations.

To take advantage of this feature, you must use the asynchronous `peformBlock:` or synchronous `peformBlockAndWait:` methods. If you're doing more complex things with your MOCs and `NSManagedObjects` within these blocks, pay attention to the documentation to ensure things don't get messy. [Here are some summary tips](http://stackoverflow.com/a/2138332/1672161).

There is no built-in callback for these methods but there are several ways to go about it, I found best suited for my purposes the `dispatch_sync` method. Two others are described in the WWDC video mentioned at the bottom of this post if your needs are different.

Flipping the switch requires getting the store and its URL, deleting the store, then creating a new store with the same URL. I learned this technique in [this post](http://stackoverflow.com/questions/1077810/delete-reset-all-entries-in-core-data), and if you hook it up to a UIButton, its a convenient way to reset things while testing.

Well, that's it, I hope this was helpful.

Oh, and if you're working with multiple asynchronous NSURLConnections, I found [this post](http://stackoverflow.com/a/332483/1672161) to be very helpful.

## References

*   WWDC 2012 Session 214 - Core Data Best Practices
*   [Apple Documentation - Dispatch Sync](http://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/Reference/reference.html#//apple_ref/c/func/dispatch_sync)