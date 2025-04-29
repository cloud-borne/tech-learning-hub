---
title: Unlock the Power of OpenAI🤖 in Google Spreadsheets📊
subtitle: Beginner's Guide to Boosting Your Data Game!

# Summary for listings and search engines
summary: Beginner's Guide to Boosting Your Data Game!

# Link this post with a project
projects: []

# Date published
date: "2024-01-16T00:00:00Z"

toc: true

# Date updated
lastmod: "2024-07-19T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- Spreadsheet
- AI

categories:

---

<!--more-->

### Overview

So, picture🖼️ this: you’re scared🫣 looking at a mountain of data in your Google Spreadsheet. It's like being lost in a forest with numbers🔢 as your only companions. Now, what if there was a nifty way to turn those cold, hard numbers into a lively conversation? Enter OpenAI🥁 Your Digital Conversationalist🤓

Integrating ```OpenAI``` with ```Google Spreadsheets``` opens up a realm of possibilities for automating and enhancing data tasks. By leveraging the capabilities of the OpenAI API, users can streamline data analysis, automate repetitive tasks, and gain deeper insights directly within their familiar spreadsheet environment.

Okay, let’s get into a bit of the nitty-gritty. Integrating OpenAI with Google Spreadsheets might sound like setting up a spaceship🚀, but it’s simpler than you think.

{{% stepper %}}
<div class="step">
  
  ## Step 1: Create an OpenAI API Key

  * Before getting started, you'll need an OpenAI API ```Key```🗝️
  * You can get one from [OpenAl's website](https://platform.openai.com/account/api-keys), which usually comes with some free🆓 credit! 

  After that, the cost is $0.002 per token. You can generate approximately 750,000 words for $2. Once you click '*Create a new secret key*' - you will only be shown it **once**, so copy and save it right away, or you'll have to create a new one.

  ![open-ai-api](/images/uploads/open-ai-api.png)

</div>
<div class="step">

  ## Step 2: Open a Google Sheet
 
  Make sure you're signed into your Google account, then visit [sheets.google.com]() and create a new blank sheet.

  This is the easy step!

</div>
<div class="step">

  ## Step 3: Create an App script.

  * Use Google Apps Script to create a custom function within your spreadsheet.
  ![app-scripts](/images/uploads/app-scripts.png)

    You should see the blank file which will open itself in a new tab:

  ![app-scripts](/images/uploads/blank-app-script.png)

</div>
<div class="step">

  ## Step 4: Create Your Apps Script Code.

  * Here's the script that fetches data from OpenAI's API using HTTP requests:

  ```js
  // Add ChatGPT Menu

  const onOpen = () => {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu("ChatGPT")
    .addItem("💾 Save Responses as Text", "saveAsText")
    .addItem("ChatGPT API Sheet from cloud-native.wiki 💜", "openUrl")
    .addToUi();
  };

  const openUrl = () => {
    const url = "https://cloud-native.wiki";
    const html = "<script>window.open('" + url + "', '_blank');google.script.host.close();</script>";
    const ui = HtmlService.createHtmlOutput(html);
    SpreadsheetApp.getUi().showModalDialog(ui, "Redirecting...");
  };

  function saveAsText(){

  // Saves formulas as plain text

  var ss = SpreadsheetApp.getActiveSheet()
  var data = ss.getDataRange().getValues()
  ss.getRange(1,1,data.length,data[0].length).setValues(data)

  }

  function CHAT(val) {
  // main function to generate the custom formula (=CHAT)
  var ss = SpreadsheetApp.getActiveSheet()

  // EITHER get API key from settings sheet
  // var setsh = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Settings");
  // var apiKey = setsh.getRange(3,1).getValue() //Cell A3

  // OR set API key
  var apiKey = YOUR_API_KEY_HERE

  // configure the API request to OpenAI
  var data = {
    "messages": [
      {"role": "system", "content": "You are an experienced content writer with high levels of expertise and authority. Your job is to write content that will be published online on websites. [Your writing style is friendly, conversational using informal grammar and sometimes non-standard English - as if you're talking to a friend, while incorporating rhetorical questions, storytelling, metaphors and analogies.] I will provide you with a topic or series of topics and you will come up with an engaging and educational copy for this topic."},

      {"role": "user", "content": "Here is your prompt:" +val +"\n"}],
    "model": "gpt-3.5-turbo",
  };

    var options = {
      'method' : 'post',
      'contentType': 'application/json',
      'payload' : JSON.stringify(data),
      'headers': {
        Authorization: 'Bearer ' + apiKey,
      },
    };

  var response = UrlFetchApp.fetch(
    'https://api.openai.com/v1/chat/completions',
    options
  );

  Logger.log(response.getContentText());

  // Send the API request 
  var result = JSON.parse(response.getContentText())['choices'][0]['message']['content'];
  Logger.log(result)

    return result

  }
  ```

</div>
<div class="step">

  ## Step 5: Deploy your changes.

  Use the blue Deploy button in the top right corner.
  ![apps-script-deploy](/images/uploads/apps-script-deploy.png)

  You should see this:
  ![new-deployment](/images/uploads/new-deployment.png)
  
</div>
<div class="step">

  ## Step 6: Add Your OpenAI API Key 
 
  There are two ways to do this, both of which are possible with the code provided before.

  * **Option 1**: On **line 29** of the Apps Script file, find ```YOUR_API_KEY``` and replace it with your API that you got in Step l.

  ![app-scripts-new](/images/uploads/app-scripts-new.png)
  
{{% callout note %}}

  If you use this method, you should ```not``` share your Google Sheet with others, as this will also share the Apps Script file that contains **your** API Key. You should not share your API key with anyone, as they can abuse all your credits and potentially cost you money💰

{{% /callout %}}

  * **Option 2**: This method is much better if you want to share your spreadsheet or create copies of it for others to use with their own API key without having to edit the code. First, create a new page within your existing sheet and call it ```Settings``` sheet.
  ![settings-sheet](/images/uploads/settings-sheet.png)

  Within that new settings page, paste your API key into the cell ```A3```, adding an optional label(s) if required:
  ![api-key-in-sheet](/images/uploads/api-key-in-sheet.png)
  Make sure this is in the ```Settings``` sheet or it won't work!

  Finally, delete the double forward slashes from the start of lines 25 and 26 or the Apps Script File and add double slashes to the start of line 29. It should look like this:
  ![api-key-in-file](/images/uploads/api-key-in-file.png)

  This is telling the script to take the API key from cell A3 of the Settings sheet that you just made.

</div>
<div class="step">

  ## Step 7: Get Prompting

  In a blank page on your sheet (NOT the settings one ... ) create two column headers called ```PROMPT``` and ```RESPONSE```.
 
  The ```PROMPT``` column is where you will type your prompts for ChatGPT and the response will appear in the ```RESPONSE``` column.
 
  In order to generate responses, type a prompt into ***Column A*** and use the formula **=CHAT(A2)** to generate a response for that prompt.

  ![prompt-response](/images/uploads/prompt-response.png)
  
  If you want to save all of your formulas as text instead of formulas so you can edit them easily, use the new button in the Google Sheets menu "Save Responses As Text"
  ![save-as-text](/images/uploads/save-as-text.png)

</div>

{{% /stepper %}}

## Final Thoughts

If you want, you can edit the contextual prompt within the Apps Script file on line 45 to better suit your needs. This is the contextual prompt used, that you can easily change:
 
The prompt reads: 

> You are an experienced content writer with high levels of expertise and authority. Your job is to write content that will be published online on websites. [Your writing style is friendly, conversational using informal grammar and sometimes non-standard English - as if you're talking to a friend, while incorporating rhetorical questions, storytelling, metaphors and analogies.] I will provide you with a topic or series of topics and you will come up with engaging and educational copy for this topic.

With this OpenAI + Google Sheets integration, you can automate **repetitive** writing tasks, **summarise** information, **plot graphs**, **statistical analysis**...the posibilities are endless.

Ready to bring these two powerhouses together? Let’s make data fun **again!** 
