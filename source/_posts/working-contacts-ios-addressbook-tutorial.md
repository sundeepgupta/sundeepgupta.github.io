---
title: 'Working with Contacts, An iOS Addressbook Tutorial'
tags:
  - ios
id: 23
categories:
  - Uncategorized
date: 2013-06-18 20:54:08
---

iOS's Addressbook framework is quite handy for enhancing the user's experience when your app requires any contact related data. In this iOS Addressbook tutorial, I detail how I use this framework in conjunction with Core Data.

<!--more-->

## Overview

This guide will explain how to:

*   Show the address book picker.
*   Store a reference to a contact.
*   Retrieve a contact using a stored reference.

It assumes prior knowledge of iOS programming and Core Data.

I had to do this for a personal project I'm working on. I didn't find any good guides, so I thought I'd write one.

References:

*   [Address Book Programming Guide for iOS](http://developer.apple.com/library/ios/#documentation/ContactData/Conceptual/AddressBookProgrammingGuideforiPhone/Introduction.html)

## Show the address book picker

We must add **AddressBook.framework** and **AddressBookUI.framework** into our project. After doing this, import the **AddressBookUI** framework and add the appropriate protocols you require into your class's .h file. For my purposes, I'm using the **ABPeoplePickerNavigationControllerDelegate** protocol.

    //  ParticipantsVC.h

    #import &lt;UIKit/UIKit.h&gt;
    #import &lt;AddressBookUI/AddressBookUI.h&gt;

    @interface ParticipantsVC : UITableViewController &lt;ABPeoplePickerNavigationControllerDelegate&gt;

    @end
    `</pre>

    Next, implement the required and optional methods defined by the protocol in the corresponding .m file. I've added a bit of code already to prevent crashing.

    <pre>`- (BOOL)peoplePickerNavigationController:(ABPeoplePickerNavigationController *)peoplePicker
          shouldContinueAfterSelectingPerson:(ABRecordRef)person {
        return NO;
    }

    - (BOOL)peoplePickerNavigationController:(ABPeoplePickerNavigationController *)peoplePicker
          shouldContinueAfterSelectingPerson:(ABRecordRef)person
                                    property:(ABPropertyID)property
                                  identifier:(ABMultiValueIdentifier)identifier {
        return NO;
    }

    - (void)peoplePickerNavigationControllerDidCancel:(ABPeoplePickerNavigationController *)peoplePicker {
        [self dismissViewControllerAnimated:YES completion:nil];
    }
    `</pre>

    Next, I setup a UI button press event to show the address book picker. We simply instantiate the picker object, set the picker's delegate property, and present it (modally).

    <pre>`- (IBAction)newButtonPress:(id)sender {
        ABPeoplePickerNavigationController *peoplePicker = [[ABPeoplePickerNavigationController alloc] init];
        peoplePicker.peoplePickerDelegate = self;

        [self presentViewController:peoplePicker animated:YES completion:nil];
    }
    `</pre>

    _No contacts?_ My simulator had an empty address book, so I added a few manually via the Contacts app. If you need to import a lot of contacts for testing, take a look at [this](http://stackoverflow.com/questions/901132/importing-addressbook-data-into-the-iphone-simulator).

    ## Store a reference to a contact

    Here is where the delegate methods come into play. I used the `-(BOOL)peoplePickerNavigationController:shouldContinueAfterSelectingPerson:` method as a handle to do things with the selected contact. Returning `YES` will push the contact detail view onto the picker's navigation stack. If you don't need that, just return `NO`.

    The most important part of this is getting the ID of the contact/person (aka ABRecordRef). To get it, use this function: `ABRecordID contactId = ABRecordGetRecordID(contact)` This will return a primitive type of `ABRecordID` which is really a 32-bit integer. Thus, your entity attribute that will hold this value should also be 32-bit integer. But don't forget before you can save it to your entity object, it needs to be wrapped in an NSNumber by calling this method: `NSNumber *contactId = [NSNumber numberWithInt:ABRecordGetRecordID(contact)]`

    A similar thing will need to happen if you want to store other properties of the ABRecordRef object. Here is an example of the contact's composite name: `NSString *contactName = (__bridge NSString *)(ABRecordCopyCompositeName(contact))`.

    In fact, [Apple suggests storing the first and last name](http://developer.apple.com/library/ios/#documentation/ContactData/Conceptual/AddressBookProgrammingGuideforiPhone/Chapters/DirectInteraction.html).

    > The recommended way to keep a long-term reference to a particular record is to store the first and last name, or a hash of the first and last name, in addition to the identifier. When you look up a record by ID, compare the record’s name to your stored name. If they don’t match, use the stored name to find the record, and store the new ID for the record.

    Once you have the ID and the other values you need to save, go ahead and store them.

    ## Retrieve a contact using a stored reference

    When it's time to use your reference to look up a contact, we first need to instantiate the address book object. I ran into a hiccup in testing for iOS 5 because the method in the docs is updated for iOS 6 only and doesn't explicitly say anything about iOS 5.

    To get the address book in a backwards compatible way, you can simply do this:

    <pre>`ABAddressBookRef addressBook;
    if (&amp;ABAddressBookCreateWithOptions != NULL) {
        addressBook = ABAddressBookCreateWithOptions(nil, nil);
    } else { //below iOS 6
        addressBook = ABAddressBookCreate();
    }
    `</pre>

    Once you have the address book object, you can use this method to retrieve a contact and any of their properties by the ID you have stored. In the code below, `participant` is a Core Data entity object.

    <pre>`ABRecordID contactID = (ABRecordID)participant.contactID.intValue;
    ABRecordRef contact = ABAddressBookGetPersonWithRecordID(addressBook, contactID);
    NSString *contactName = (__bridge NSString *)ABRecordCopyCompositeName(contact);

## Conclusion

I hope this makes things a bit easier for anyone starting to dig into the address book API.