---
layout: post
title:  "PDF creation service with Google Apps Script"
date:   2015-5-14
categories: projects
---

Automatically generating PDF's from a template is no small feat without the proper tools, but Google Apps Script makes it simple. In less than 50 lines of JavaScript we can create an API that generates PDF's based on a template in our Google Drive.

Use cases are endless, but my current company uses it for generating invoices on demand from my [Chrome Extension](http://www.forrestbradley.com/projects/featured/2015/12/16/salesforce-macros-part1.html).

To get started, sign in to Google Drive and create a new Google Apps Script. If you haven't done this before, you'll have to connect it by searching under **New** > **More** > **Connect more apps**.

Once you're inside the editor, we can start building the API. Our API will only have one method: GET. Since Apps Script maps requests to function names, we'll call our first function as mandated in the docs: doGet.

{% highlight js %}
function doGet(e) {
  var output = {};

  createInvoice(e.parameter, function(invoice) {
    output = {
      id: invoice.getId(),
      url: invoice.getUrl()
    }
  });

  return ContentService.createTextOutput(JSON.stringify(output))
    .setMimeType(ContentService.MimeType.JSON);
}
{% endhighlight %}

We can extract the URL parameters from e.parameters. After generating the PDF, we'll return a download link and other information inside a JSON object. Now let's add the functions that do the real work.

{% highlight js %}
// Replace below with your template's ID. You can find this in the URL
var SOURCE_TEMPLATE = "123asdf_YOUR_DOCUMENT_ID_GOES_HERE_123asdf";

// Replace below with the folder ID. Generated PDF's will be stored there
var TARGET_FOLDER = "123asdf_YOUR_FOLDER_ID_GOES_HERE_123asdf";

/**
 * Duplicates a Google Apps doc
 *
 * @return a new document with a given name from the orignal
 */
function createDuplicateDocument(sourceId, name) {
    var source = DriveApp.getFileById(sourceId);
    var targetFolder = DriveApp.getFolderById(TARGET_FOLDER);
    var newFile = source.makeCopy(name, targetFolder);

    return DocumentApp.openById(newFile.getId());
}

/**
 * Search a paragraph in the document and replaces it with the generated text 
 */
function replaceText(doc, parameters) {
  var body = doc.getBody();
  for (param in parameters) {
    body.replaceText("<" + param + ">", parameters[param]);
  }

  doc.saveAndClose();
}

/**
 * Script entry point
 */
function createInvoice(parameters, callback) {
  var target = createDuplicateDocument(SOURCE_TEMPLATE, "Billing Summary - " + parameters.name);

  Logger.log("Created new document:" + target.getId());

  replaceText(target, parameters);

  callback(target);
}
{% endhighlight %}

The template file will have <variables> that the above script will replace. For example, if our template contains the text <price>, it will be swapped out with the price URL parameter.
