1. `manifest.json`: This file will contain the configuration and permissions for the Chrome Manifest V3 extension.

### Structure of Generated Code:

#### `manifest.json`:

The `manifest.json` file will have the following structure:

`````json
{
  "manifest_version": 3,
  "name": "Anthropic Claude Extension",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "32": "images/icon32.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "icons": {
    "16": "images/icon16.png",
    "48": "images/icon48.png",
    "128": "images/icon128.png"
  }
}

````md
#### `background.js`:

The `background.js` file will handle the communication between the content script and the popup script. It will have the following structure:

```javascript
// Listen for messages from the content script
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "storePageContent") {
    // Store the page content in local storage
    chrome.storage.local.set({ pageContent: request }, () => {
      sendResponse({ success: true });
    });
  } else if (request.action === "getPageContent") {
    // Retrieve the page content from local storage
    chrome.storage.local.get(["pageContent"], (result) => {
      sendResponse(result);
    });
  }
  return true;
});

// TODO: Handle API key storage and retrieval

// TODO: Handle Anthropic API call

// TODO: Handle clearing API key on 401 error
`````

#### `popup.html`:

The `popup.html` file will contain the HTML structure of the popup UI. It will have the following structure:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Anthropic Claude Extension</title>
    <link rel="stylesheet" href="https://unpkg.com/mvp.css@1.12/mvp.css" />
    <style>
      /* Additional custom styles for the popup */
      /* ... */
    </style>
    <script src="popup.js"></script>
  </head>
  <body>
    <div id="loadingIndicator"></div>
    <h1 id="pageTitle"></h1>
    <form>
      <label for="userPrompt">User Prompt:</label>
      <textarea id="userPrompt" rows="2"></textarea>
      <label for="stylePrompt">Style Prompt:</label>
      <textarea id="stylePrompt" rows="4"></textarea>
      <input type="submit" id="sendButton" value="Submit" />
    </form>
    <div id="content"></div>
  </body>
</html>
```

#### `popup.js`:

The `popup.js` file will handle the functionality of the popup UI. It will have the following structure:

```javascript
// Get DOM elements
const loadingIndicator = document.getElementById("loadingIndicator");
const pageTitle = document.getElementById("pageTitle");
const userPrompt = document.getElementById("userPrompt");
const stylePrompt = document.getElementById("stylePrompt");
const sendButton = document.getElementById("sendButton");
const content = document.getElementById("content");

// Set default values for prompts
const defaultPrompt = `Please provide a detailed, easy to read HTML summary of the given content`;
const defaultStyle = `Respond with 3-4 highlights per section with important keywords, people, numbers, and facts bolded in this HTML format:
...
`;

// Function to request a summary from the Anthropic API
function requestAnthropicSummary(prompt, apiKey) {
  // TODO: Implement Anthropic API call
}

// TODO: Add event listeners and handle form submission

// TODO: Retrieve stored API key from local storage

// TODO: Display loading indicator while waiting for API response

// TODO: Retrieve page content from background script

// TODO: Display page title

// TODO: Handle form submission and request summary from Anthropic API

// TODO: Display Anthropic-generated result
```

### Conclusion:

The generated code will include the necessary files and structures to create a Chrome Manifest V3 extension. The `manifest.json` file will contain the extension configuration, while the `background.js` file will handle communication between the content script and the popup script. The `popup.html` file will define the HTML structure of the popup UI, and the `popup.js` file will handle the functionality of the popup UI, including retrieving page content, requesting summaries from the Anthropic API, and displaying the results.
