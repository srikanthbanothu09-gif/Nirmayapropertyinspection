# Google Sheets Integration Guide for NIRMAYA Property Inspection

This guide explains how to connect your website forms (both the **Property Inspection Request** on the home page and the **Contact Form** on the contact page) to your Google Sheet:
`https://docs.google.com/spreadsheets/d/17pOVPQRSWFEp4GogMe4nEuK8hIEpDizLEszqj_fbDG4/edit?usp=sharing`

We will use a free, robust, and serverless **Google Apps Script** to handle form submissions and automatically append rows to your spreadsheet.

---

## Step 1: Open Apps Script in Your Google Sheet

1. Open your Google Sheet in your web browser:
   [NIRMAYA Leads Spreadsheet](https://docs.google.com/spreadsheets/d/17pOVPQRSWFEp4GogMe4nEuK8hIEpDizLEszqj_fbDG4/edit)
2. In the top menu, click on **Extensions** -> **Apps Script**.
3. A new tab will open with a code editor, usually containing a blank function named `myFunction()`.

---

## Step 2: Paste the Integration Script

1. Select everything in the Apps Script editor and delete it.
2. Copy and paste the following Google Apps Script code into the editor:

```javascript
/**
 * NIRMAYA Property Inspection - Form Submission Handler
 * This script runs in Google Sheets and appends new form submissions as rows.
 */

function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  // Parse the incoming JSON data
  var data;
  try {
    if (e.postData && e.postData.contents) {
      data = JSON.parse(e.postData.contents);
    } else {
      data = e.parameter;
    }
  } catch (err) {
    data = e.parameter;
  }
  
  // Return error if data is empty
  if (!data || Object.keys(data).length === 0) {
    return ContentService.createTextOutput(JSON.stringify({ status: "error", message: "No data received" }))
                         .setMimeType(ContentService.MimeType.JSON)
                         .setHeader("Access-Control-Allow-Origin", "*");
  }
  
  var timestamp = data.submittedAt || new Date().toISOString();
  var formType = data.formType || "unknown";
  
  // Define standard columns
  var headers = [
    "Timestamp", 
    "Form Type", 
    "Name", 
    "Email", 
    "Phone", 
    "State", 
    "Location", 
    "Property Address", 
    "Property Type", 
    "Inspection Type", 
    "Preferred Date", 
    "Message", 
    "Lead ID"
  ];
  
  // Initialize headers if sheet is empty
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(headers);
  } else {
    // Read existing headers to map fields correctly (in case headers are in a different order)
    var existingHeaders = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    if (existingHeaders.length === 1 && existingHeaders[0] === "") {
      sheet.appendRow(headers);
    } else {
      headers = existingHeaders;
    }
  }
  
  // Map data fields to match columns
  var row = [];
  for (var i = 0; i < headers.length; i++) {
    var header = headers[i].toString().toLowerCase().trim();
    var value = "";
    
    if (header.includes("timestamp") || header.includes("date/time") || header.includes("submitted")) {
      value = timestamp;
    } else if (header.includes("form") && header.includes("type")) {
      value = formType;
    } else if (header.includes("name") || header.includes("full name")) {
      value = data.fullName || data.name || "";
    } else if (header.includes("email")) {
      value = data.email || "";
    } else if (header.includes("phone")) {
      value = data.phone || data.phoneNumber || "";
    } else if (header.includes("state")) {
      value = data.state || "";
    } else if (header.includes("location") || header.includes("city")) {
      value = data.location || "";
    } else if (header.includes("property address") || header.includes("site address") || header.includes("address")) {
      value = data.propertyAddress || data.address || "";
    } else if (header.includes("property type")) {
      value = data.propertyType || "";
    } else if (header.includes("inspection type")) {
      value = data.inspectionType || "";
    } else if (header.includes("preferred date") || header.includes("slot")) {
      value = data.preferredDate || "";
    } else if (header.includes("message") || header.includes("instruction") || header.includes("comment")) {
      value = data.message || "";
    } else if (header.includes("id") || header.includes("lead")) {
      value = data.id || "";
    } else {
      // Direct property matching if header name matches an incoming data key
      var keyFound = Object.keys(data).find(function(k) {
        return k.toLowerCase() === header;
      });
      if (keyFound) {
        value = data[keyFound];
      }
    }
    row.push(value);
  }
  
  // Append new row to Google Sheets
  sheet.appendRow(row);
  
  // Send email notification to owner
  try {
    var emailRecipient = "nirmayapropertyinspection@gmail.com";
    var subject = "New NIRMAYA Lead: " + (formType === "inspection" ? "Inspection Request" : "Contact Message");
    var htmlBody = "<h3>New NIRMAYA Lead Received</h3>" +
                   "<p><strong>Form Type:</strong> " + (formType === "inspection" ? "Property Inspection" : "General Contact") + "</p>" +
                   "<table border='1' cellpadding='5' style='border-collapse: collapse; font-family: Arial, sans-serif;'>" +
                   "<tr style='background-color: #f2f2f2;'><th>Field</th><th>Value</th></tr>";
    
    for (var i = 0; i < headers.length; i++) {
      htmlBody += "<tr><td><strong>" + headers[i] + "</strong></td><td>" + row[i] + "</td></tr>";
    }
    htmlBody += "</table><br><p>This is an automated email from your Google Apps Script integration.</p>";
    
    MailApp.sendEmail({
      to: emailRecipient,
      subject: subject,
      htmlBody: htmlBody
    });
  } catch (emailErr) {
    console.error("Email notification failed: " + emailErr.toString());
  }
  
  // Return success JSON response
  return ContentService.createTextOutput(JSON.stringify({ status: "success", rowAdded: row }))
                       .setMimeType(ContentService.MimeType.JSON)
                       .setHeader("Access-Control-Allow-Origin", "*");
}

function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({ status: "running", message: "NIRMAYA Google Sheets Web App is active!" }))
                       .setMimeType(ContentService.MimeType.JSON)
                       .setHeader("Access-Control-Allow-Origin", "*");
}
```

3. Click the **Save** icon (disk icon) or press `Ctrl + S` to save the script. You can rename the script project to **NIRMAYA Sheets Integration** by clicking on "Untitled project" at the top-left.

---

## Step 3: Deploy as a Web App

Google Apps Script needs to be deployed as a Web App so the website can communicate with it.

1. In the top-right corner of the Apps Script window, click the **Deploy** button and select **New deployment**.
2. In the configuration dialog, click the gear icon next to **Select type** and select **Web app**.
3. Configure the following settings:
   - **Description:** `NIRMAYA Form Submissions Web App v1`
   - **Execute as:** `Me (your-email@gmail.com)` (This allows the script to write to *your* spreadsheet using your permissions).
   - **Who has access:** `Anyone` (This is **CRITICAL** - it allows the website form to submit data without prompting users to log in).
4. Click **Deploy**.
5. *Note:* If this is your first time deploying, Google will ask you to **Authorize Access**. Click **Authorize access**, log in with your Google account, click **Advanced** (on the warning screen), and click **Go to NIRMAYA Sheets Integration (unsafe)**, then click **Allow**.
6. Once authorization is complete, you will see a screen with the **Web app URL**.
7. **Copy this URL** (it should look like `https://script.google.com/macros/s/XXXXXX/exec`).

---

## Step 4: Configure the Website to Use Your Web App

Now we connect the website to the Google Apps Script Web App:

1. In your project codebase, open the [js/config.js](file:///c:/Users/srikanth%20Banothu/Desktop/GEN%20AI%20tools/New%20folder/js/config.js) file.
2. Locate the line for `GOOGLE_SHEET_SCRIPT_URL`:
   ```javascript
   GOOGLE_SHEET_SCRIPT_URL: "",
   ```
3. Paste the copied Web App URL inside the quotation marks, like so:
   ```javascript
   GOOGLE_SHEET_SCRIPT_URL: "https://script.google.com/macros/s/YOUR_ACTUAL_DEPLOED_SCRIPT_ID/exec",
   ```
4. Save the file.

---

## Testing Your Setup

1. Start your local server (`npm start` or double-click `index.html` to open in browser).
2. Fill out the **Request Property Inspection** form on the homepage and submit it.
3. Check your Google Sheet. You should instantly see a new row populated with all your inputs!
4. Do the same with the **Send a Message** form on the Contact page, and verify the data gets recorded correctly under the `Form Type: contact` column.
