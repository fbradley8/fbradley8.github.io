---
layout: post
title:  "Automating Salesforce for Operations"
date:   2015-12-16
categories: projects featured
---
Before macros were built into Service Cloud and other editions, Salesforce Enterprise was (and still is) a stubborn pig that made automation a pain in the ass. Mail merge had limitations that made the feature useless altogether.

Our registration emails needed to include data from an account's opportunities and cases, but mail merge will only traverse up the inheritance tree and not back down. In order to pull fields across objects I would have to create a solution that existed outside of Salesforce. After doing some research into what tools were available I settled on the idea of creating a Chrome extension. The extension would house an Angular application using JSforce for data calls.

Now that we have the technologies sorted out let's get to features.

#####Email Macros

How many times a day do you send out the same email? Today was a great day and you onboarded 20 new clients. The steps to send out this typical email are:

* Go to the case you're working out of
* Click Send Email
* Select the email your department uses for the From field
* Open a new tab and go back to the account to see if the To field has the correct billing contact
* Pick the email template
* Change the subject if needed
* Attach a copy of the client's first invoice that you built by hand

We'll dive into that last step later, but the others can all be predefined and filled out by a macro.

IMAGE

The user of this extension will create a new macro and map email fields to either object fields or text. For example, this macro will always use the same email template, so I will map the templateId email field to the ID of the template I need, stored as text. Other email fields are mapped to case and account fields.

Now, when the user opens the Chrome extension and clicks the onboarding email macro, the email template will automatically be selected and all of the fields will be prefilled. All the user has to do is click send.

#####Document Generation

This part will seem crazy and won't apply to everyone, but the concept is what matters. I used to build invoices by hand in Powerpoint, pulling billing data from opportunities. It would then get exported as a PDF and attached to onboarding emails in Salesforce. You would be hard pressed to find a company that onboards over 20 clients per day and builds invoices manually. Regardless, let's see how I handled this.

This would be a new type of macro. To save time, I actually cheated and hard-coded this macro. There was no point in adding the functionality to script macros. It would reduce user friendliness and only benefit power users, of which our client services department had none. I named the macro "Generate Invoice" and when clicked, the following events would execute:

* Get the object the user is looking at using chrome.tabs api
* After looking up the object, lookup the associated account
* Retreive the desired opportunity based on type and closed/won
* POST the data to a Google Apps Script that will assemble a document and return it as a PDF

Read about how to create a document-generating Google Apps Script in part 2 (coming soon).
