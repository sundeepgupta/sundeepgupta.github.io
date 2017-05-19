---
title: Writing Clean Functions
tags:
  - coding
id: 14
categories:
  - Uncategorized
date: 2013-06-07 12:51:05
---

After reading some more of the book Clean Code, I was excited to try things out and demonstrate writing clean functions.

<!--more-->

I read part way through Chapter 3 on Functions today from the book [Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882). I think the biggest take away from writing functions is to keep them absolutely tiny. What stuck into my brain is this:

1.  The function should have a pure and single purpose.
2.  The purpose should be clearly stated via the function name
3.  The function's code should be made up of only one level of abstraction below the function's level of abstraction.

I think 1 and 2 are pretty clear, but 3 is something new to me. I wanted to try this on a real example of my own code (in Objective-C) to fully absorb this. Here is the before:

    - (void)emailOrder
    {
        MFMailComposeViewController *mailer = [[MFMailComposeViewController alloc] init];
        mailer.mailComposeDelegate = self;

        //Get the user's company name
        NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
        NSDictionary *companyInfo = [defaults objectForKey:USER_COMPANY_INFO];
        NSString *userCompanyName = companyInfo[USER_COMPANY_NAME];

        NSString *subject = [NSString stringWithFormat:@"Order #%@ from %@", self.order.scOrderId, userCompanyName];
        NSString *fileName = [NSString stringWithFormat:@"Order %@", self.order.scOrderId];
        NSString *body = [NSString stringWithFormat:@"Dear %@,\n\nA copy of your order is attached to this email. We appreciate your business, thank you.\n\n", self.order.customer.dbaName];

        NSString *pdfPath = [self pathForFileName:PDF_FILENAME withFileNameExtension:PDF_FILENAME_EXTENSION];
        if (pdfPath) {
            NSData *pdfData = [NSData dataWithContentsOfFile:pdfPath];
            [self.webView loadData:pdfData MIMEType:PDF_MIME_TYPE textEncodingName:PDF_TEXT_ENCODING baseURL:nil];
            [mailer addAttachmentData:pdfData mimeType:PDF_MIME_TYPE fileName:fileName];
        }

        [mailer setSubject:subject];

        //Get the email objects, then build array with the actual email address strings
        NSMutableArray *emailStrings = [[NSMutableArray alloc] init];
        for (SCEmail *email in self.order.customer.emailList.allObjects) {
            [emailStrings addObject:email.address];
        }

        [mailer setToRecipients:emailStrings];
        if ([companyInfo objectForKey:USER_COMPANY_EMAIL]) {
            NSArray *ccArray = [NSArray arrayWithObject:[companyInfo objectForKey:USER_COMPANY_EMAIL]];
            [mailer setCcRecipients:ccArray];
        }

        [mailer setMessageBody:body isHTML:NO];
        [self presentViewController:mailer animated:YES completion:nil];
    }
    `</pre>

    The code has several things going on and certainly goes several levels of abstraction deep. So lets break it down using the method in the book. I'll only go down one branch of the hierarchy.

*   To email the order, we setup the mailer and we display it on screen.
        *   To setup the email composer, we instantiate the emailer object and set its properties.
            *   To set the mailer's properties, we setup the delegate and we setup the recipients and we setup the subject and we setup the body and we setup the attachments.
                *   To setup the recipients, we setup the "to" recipients and we setup the "cc" recipients.
                    *   To setup the "to" recipients, we create an array and we fill the array with the email addresses and we set the mailer's property with that array.

    Here is the after:

    <pre>`- (void)emailOrder
    {
        MFMailComposeViewController *mailer = [self setupMailer];
        [self presentViewController:mailer animated:YES completion:nil];
    }

    - (MFMailComposeViewController *)setupMailer
    {
        MFMailComposeViewController *mailer = [[MFMailComposeViewController alloc] init];
        [self setupPropertiesforMailer:mailer];
        return mailer;
    }

    - (void)setupPropertiesforMailer:(MFMailComposeViewController *)mailer
    {
        mailer.mailComposeDelegate = self;
        [self setupRecipientsForMailer:mailer];
        [self setupSubjectForMailer:mailer];
        [self setupBodyForMailer:mailer];
        [self setupAttachmentForMailer:mailer];
    }

    - (void)setupRecipientsForMailer:(MFMailComposeViewController *)mailer
    {
        [self setupToRecipientsForMailer:mailer];
        [self setupCcRecipientsForMailer:mailer];
    }

    - (void)setupToRecipientsForMailer:(MFMailComposeViewController *)mailer
    {
        NSMutableArray *emailStrings = [[NSMutableArray alloc] init];
        for (SCEmail *email in self.order.customer.emailList.allObjects) {
            [emailStrings addObject:email.address];
        }
        [mailer setToRecipients:emailStrings];
    }

    - (void)setupCcRecipientsForMailer:(MFMailComposeViewController *)mailer
    {
        NSDictionary *userCompanyInfo = [self fetchUserCompanyInfo];
        if ([userCompanyInfo objectForKey:USER_COMPANY_EMAIL]) {
            NSArray *ccArray = [NSArray arrayWithObject:[userCompanyInfo objectForKey:USER_COMPANY_EMAIL]];
            [mailer setCcRecipients:ccArray];
        }
    }

    - (void)setupSubjectForMailer:(MFMailComposeViewController *)mailer
    {
        NSDictionary *userCompanyInfo = [self fetchUserCompanyInfo];
        NSString *userCompanyName = userCompanyInfo[USER_COMPANY_NAME];
        NSString *subject = [NSString stringWithFormat:@"Order #%@ from %@", self.order.scOrderId, userCompanyName];
        [mailer setSubject:subject];
    }

    - (void)setupBodyForMailer:(MFMailComposeViewController *)mailer
    {
        NSString *body = [NSString stringWithFormat:@"Dear %@,\\n\\nA copy of your order is attached to this email. We appreciate your business, thank you.\\n\\n", self.order.customer.dbaName];
        [mailer setMessageBody:body isHTML:NO];
    }

    - (void)setupAttachmentForMailer:(MFMailComposeViewController *)mailer
    {
        NSString *fileName = [NSString stringWithFormat:@"Order %@", self.order.scOrderId];
        NSString *pdfPath = [self pathForFileName:PDF_FILENAME withFileNameExtension:PDF_FILENAME_EXTENSION];
        if (pdfPath) {
            NSData *pdfData = [NSData dataWithContentsOfFile:pdfPath];
            [self.webView loadData:pdfData MIMEType:PDF_MIME_TYPE textEncodingName:PDF_TEXT_ENCODING baseURL:nil];
            [mailer addAttachmentData:pdfData mimeType:PDF_MIME_TYPE fileName:fileName];
        }
    }

    - (NSDictionary *)fetchUserCompanyInfo
    {
        NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
        return [defaults objectForKey:USER_COMPANY_INFO];
    }

It took some time to refactor it, and it still needs a clean up on the names, but now I have a good idea of the concept. It felt like I was thinking more like a robot - very logical and procedural thinking. One observation I have is that the code is definitely more readable now. I can tell what a function is doing now at a glance. The only exception however are the leaf functions in the hierarchy. Because the code in those are not made up of more function calls, the language syntax makes it less readable.