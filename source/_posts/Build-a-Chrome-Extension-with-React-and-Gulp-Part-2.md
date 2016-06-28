---
title: Build a Chrome Extension with React and Gulp - Part 2
date: 2016-06-27 17:16:29
categories:
- Development

tags:
- JavaScript
- Gulp
- Chrome
- React
---

In this part I will focus on the the two part of this extension:
1. Show a popup window when user clicks the extension icon
2. Messaging between UI component and content script

For the basics of building an extension please refer to part 1: [Build a Chrome Extension with React and Gulp - Part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/).

<!-- more -->

## 1. Popup menu window
We need a function to show a menu window in the top-right corner of page when user clicks the extension icon.
### a. Get the event in the background script
Because only the script in background layer lives in the whole browser lifecycle and can handle some specified browser event such like icon clicked event. Thus we need to put our icon event handler in a background script.

{% codeblock lang:js background.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/background/background.js background.js %}
chrome.browserAction.onClicked.addListener(function (tab) { 
    //Fired when User Clicks ICON
    // Then insert a content script file into current page
    chrome.tabs.executeScript(null, { file: "scripts/content_scripts/inject_menu.js", allFrames: true });
});
{% endcodeblock %}


After the above code fired and executed, a content script `inject_menu.js` will be inserted into the current page. As we mentioned in [Part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/), background script cannot manipulate the DOM directly because it doesn't share the context with page, that is why we need to inject a content script into the page. Let's see what the `inject_menu.js` do:


{% codeblock lang:js inject_menu.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/content_scripts/inject_menu.js inject_menu.js %}
var extensionOrigin = 'chrome-extension://' + chrome.runtime.id;
var id = chrome.runtime.id;

// Test if menu window has already been injected
var ifWindowExists = function () {
    return document.getElementById(id) != null;
}

if (!location.ancestorOrigins.contains(extensionOrigin)) {
    var iframe;

    if (!ifWindowExists()) {
        iframe = document.createElement('iframe');
        // Must be declared at web_accessible_resources in manifest.json
        iframe.src = chrome.runtime.getURL('/html/menu.html');
        iframe.id = chrome.runtime.id;
        iframe.dataset.shown = false;
        iframe.style.cssText = 'position:fixed;top:0;right:0;display:block;' +
            'width:0;height:0;z-index:2999999999;border:none;';
        document.body.appendChild(iframe);  // Injected!
    }else{
        iframe = document.getElementById(id);
    }
    
    // Show the window only when it is ready
    if (iframe.dataset.shown == "false" && iframe.dataset.ready == "true") {
        window.parent.postMessage({ application: 'video_caption', type: "UI_SHOW", message: "" }, "*");
    }else if(iframe.dataset.shown == "true"){
        window.parent.postMessage({ application: 'video_caption', type: "UI_HIDE", message: "" }, "*");
    }
}
{% endcodeblock %}

Now, we already inject our iframe into the current page but you may notice that the width and height of the iframe is 0, that is because we want to show the window with animataion when the iframe is ready and also hide the menu when user clicks the icon again. However, you have already known how to inject something into the page so next let's see how to communicate between page component and content script.

### 2. Messaging between page component(popup menu window) and content script
Once we inject the menu window to the page, we want to do two things:
1. Send an event to content script when the menu window to notify the menu window is ready to show contents
2. Content script shows the menu window iframe with animation when it receives ready event

To achieve this, let's first look at the menu iframe we just injected into the page:

{% codeblock lang:html menu.html https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/html/menu.html menu.html %}
<!doctype html>
<html>
<head>
  <link type="text/css" rel="stylesheet" href="/styles/menu.css">
</head>

<body>
  <!-- React root loaded here -->
  <div id="application"></div>
</body>

<!-- build:react  -->
<script src="/public/react/react.js" type="text/javascript"></script>
<script src="/public/react/react-dom.js" type="text/javascript"></script>
<!-- endbuild -->

<script src="/public/jquery-2.2.4.min.js" type="text/javascript"></script>
<script src="/scripts/menu.app.js" type="text/javascript"></script>
</html>
{% endcodeblock %}

The html is really simple, all of the UI component is written in React so the idea is to send the message from UI component to content script in the `componentDidMount` event of the menu React component, the code looks like:

{% codeblock lang:js inject_menu.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/components/menu.react.jsx menu.react.jsx %}
var React = require('react');
var ReactDOM = require('react-dom');
// Loading indicator UI component
var LoadingIndicator = React.createClass({ });

// Close button UI component
var CloseButton = React.createClass({ });

// Menu warning message UI component
var MenuMessage = React.createClass({ });

// Video information UI component
var VideoInformation = React.createClass({ });

// Menu control UI component
var CaptionControl = React.createClass({ });

// Menu main window UI component
var MenuWindow = React.createClass({
    getInitialState: function () {
        return { loaded: false, metadata: null, message: '' };
    },
    handleClose: function () { window.parent.postMessage({ application: 'video_caption', type: "UI_HIDE", message: "" }, "*"); },
    componentDidMount: function () {
        var that = this;
        // Send ready message to content-script
        window.addEventListener("message", function (event) {
            if (event.data.application == "video_caption" && event.data.type == "UI_INIT") {
                var metadata = null;
                var message = "";
                if (event.data.success != false) {
                    metadata = JSON.parse(event.data.message);
                }
                else {
                    message = event.data.message;
                }
                that.setState({ "loaded": true, "metadata": metadata, "message": message });
            }
        }, false);
        // Send a message to content script
        window.parent.postMessage({ application: 'video_caption', type: "UI_READY", message: "" }, "*");
    },
    render: function () {
        return (
            <div id="main-container">
                <CloseButton onClose={this.handleClose}/>
                <LoadingIndicator loaded={this.state.loaded} />
                <div id="content-container">
                    <h1 id="project-title">Caption Project</h1>
                    <MenuMessage message={this.state.message} />
                    <VideoInformation metadata={this.state.metadata} />
                    <CaptionControl metadata={this.state.metadata} />
                </div>
            </div>
        );
    }
});

module.exports = MenuWindow;
{% endcodeblock %}
 
I leave out a lot of components code for simplicity and there are three things you need to be aware:
1. I call `postMessage` function on `window.parent` instead of `window`, it is because our content script share the context with the main page not the menu window iframe, so all of the message should pass from main page to extension and vice verse. 
2. Only content script has the ability to manipulate the DOM which means you can only put the function which show the menu window with animation in the content script but not in the iframe script.
3. From line 27-39, I register an event handler to receive the UI_INIT message from content script because all of the data is retrieve by content script and pass to UI component. So the message flow looks like：
    `UI Component -> message(UI_Ready) -> Event Handler(Fetch data and do something) -> message(UI_INIT) -> UI Component -> show UI`

Now it is time to look at the message handler, I create a new content script `event_handler.js` to handle all messages from background or UI component. 

{% codeblock lang:js event_handler.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/content_scripts/event_handler.js event_handler.js %}
/**
 * Show menu UI with animation
 * @author Junkai Yang 
 */
var showUI = function () {
    $("#" + chrome.runtime.id).velocity({ width: 340, height: 400 }, 250);
    $("#" + chrome.runtime.id)[0].dataset.shown = true;

}
/**
 * Hide menu UI with animation
 */
var hideUI = function () {
    $("#" + chrome.runtime.id).velocity({ width: 0, height: 0 }, 250);
    $("#" + chrome.runtime.id)[0].dataset.shown = false;
}


/**
 * Invoked by event handler when ui ready message is received.
 * Get the video information and retrieve the captions from caption server then pass them to the menu frame dom.
 * 
 * @author Junkai Yang
 * @param  {dom} frame - menu frame dom element
 * @param  {dom} videoElement - video dom element
 */
var initUI = function () {
    // Show UI if ready
    if (!frame.dataset.ready && frame.dataset.shown != "true") {
        showUI();
    }
    frame.dataset.ready = "true";
}

// Add listerners to window message system
window.addEventListener("message", function (event) {
    // Check the message sender
    if (event.data.application != 'video_caption')
        return;
    else {
        // Receive a message from our extension
        console.log("Content script received: " + event.data.type);
    }

    switch (event.data.type) {
        case "UI_HIDE":
            hideUI();
            break;
        case "UI_SHOW":
            showUI();
            break;
        case "UI_READY":
            initUI();
            break;
    }
}, false);
{% endcodeblock %}
 
## 3. Summary
So far, we have already create these things:
1. `background.js`: runs a content script `inject_menu.js` when user clicks the extension icon
2. `inject_menu.js`: inject an iframe into the page 
3. `menu.html`: the html file for menu window
4. `menu.react.jsx`: the script file for `menu.html`, send `UI_READY` message to `event_handler.js` when it is ready and receive the `UI_INIT` message from `event_handler.js` to initialize the UI component
5. `event_handler.js`: a content script which handle all message from UI component    

Now if we put all these codes together and you can see when we click the extension icon then a menu window will be shown on the top-right corner of this page. You can check all the codes in my repo [here](https://github.com/cyfloel0516/video_caption_chrome_extension). **But pay attention that my code doesn't work on any website because it is just for test and only works for some YouTube and Coursera page.** However, it should be easy to pull out some codes and use them in your project.

[Build a Chrome Extension with React and Gulp - Part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-1/)
[Build a Chrome Extension with React and Gulp - Part 2](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/)