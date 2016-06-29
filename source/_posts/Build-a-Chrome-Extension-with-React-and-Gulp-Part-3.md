---
title: Build a Chrome Extension with React and Gulp - Part 3
date: 2016-06-27 22:52:58
categories:
- Development

tags:
- JavaScript
- Gulp
- Chrome
- React
---
In last two posts I demostrate some basics and key points of Chrome extension development and also show some basic steps of building an extension. In this post, I will elaborate on messaging because this part has the biggest difference compared to a traditional website. 

In a website, you can do everything you want with one script and simply call a JavaScript function to do what you want. However, as I said before in the [part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-1/) each layer in extension can only do their own job and you need messaging between different layer to achieve your final goal. 

So, let's see how different types of messaging together to make a complete Chrome extension works.

<!-- more --> 
## 1. Types of messaging in Chrome extension
There are three types of messaging in Chrome extension:
1. Messaging between UI component and extension
2. Messaging within extension
3. Messging between extension and native application

Each type of messaging has its different role in the extension and next I will go though each type in detail.

## 2. Messging between UI component and extension
Messaging between UI component is frequently used in extension development because it is the only way to implement the communication between page and extension. The content script and page shares the same context which means you can use `window.postMessage` function to exchange message between page and extension.

Here is an example for you to understand when to use this type of messaging works: 
1. UI component -> Extension(Content Script)
We create a popup window menu in [Part 2](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/). The window looks like: 
![](menu_window.png)

So when the user clicks the `Load Caption` button we need to load the captions to YouTube player but the barriers are:
1. The window is an iframe injected into the origin page by content script, so the window cannot manipulate the origin page to load the captions
2. The iframe cannot call any functions in extension because once the menu iframe is injected into the page, it behaves just like an origin page

That's why we need to send a message from the window iframe to the content script in extension.

2. Extension(Content Script) -> UI Component
After loading the captions on YouTube player, we need to update the sentences with the video playing. We can either manipulate the sentence directly in content script or we can also send a message to the UI component to notify an update. The second way is an example to show when we need this type of messaging

Now, let's look at the code in the UI compoent to send the message.

{% codeblock lang:js menu.react.jsx  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/components/menu.react.jsx menu.react.jsx %}
// Menu control UI component
var CaptionControl = React.createClass({
    handleLoadCaption: function () {
        window.parent.postMessage({ application: 'video_caption', type: "LOAD_CAPTION", message: "" }, "*");
    },
    handleOpenEditor: function () {
        window.parent.postMessage({ application: 'video_caption', type: "OPEN_EDITOR", message: "" }, "*");
    },
    render: function () {
        return (
            <div id="control-container" >
                <button id="load-caption" className="btn btn-success" onClick={this.handleLoadCaption}>Load Caption</button>
                <button id="load-caption" className="btn btn-success" onClick={this.handleOpenEditor}>Open Caption Editor</button>
            </div>
        )
    }
});
{% endcodeblock %}

In the receiver side: 
{% codeblock lang:js event_handler.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/content_scripts/event_handler.js event_handler.js %}
/**
 * Invoked by event handler when load caption event is received.
 * It attach a caption dom element to the video element and change it dynamically.
 * 
 * @author Junkai Yang
 * @param  {dom} frameDom
 * @param  {dom} videoDom
 */
var loadCaption = function (frameDom, videoDom) {
    var $caption = $("<p>caption sentence</p>");
    // Append it to the video player
    $(VideoApi.videoContainerSelector).append($caption);
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
        // Other events
        ...
        ...
        case "LOAD_CAPTION":
            loadCaption(frame, video);
            break;
    }
}, false);
{% endcodeblock %}

**Be care, there is a subtle difference if you want to send message from `Content Script` to `UI component`. You can't simply send message to the parent page, instead you have to send message to the iframe**. See the following code, you have to get the DOM element of the iframe and then send the message. If you send the message to the window, the child iframe will never receive the it.

{% codeblock lang:js event_handler.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/content_scripts/event_handler.js event_handler.js %}
/**
 * Send message to editor to update the caption
 * @param  {object} caption
 */
var updateCaption = function (caption) {
    document.getElementById("video_caption_editor_" + chrome.runtime.id).contentWindow.postMessage({
        application: "video_caption", type: "UPDATE_CAPTION", message: {
            caption: captions[0]
        }
    }, "*");
}
{% endcodeblock %}

## 3. Messging winthin extension
It seems weird why we need to communicate within extension, but just take a look at this case:

Take the menu window for example, when user clicks the `Open Editor` button, we need to inject another iframe(the editor) into the page but for the neat of our project structure, we create a new content script file to do this job instead of putting all code together in a same content script. Then, when user clicks the button, the message first send from `menu UI component` to `event_handler.js(Content Script)`, then the message should pass from `event_handler.js(Content Script)` to `background.js`. At last, the background should call `inject_editor.js(Content Script)` to do the job of opening the editor.

I will not show the messaging between `menu UI component` and `event_handler.js` because the code is almost same as which in the first type. The most obvious difference is that we use Chrome messaging API instead of window.postMessage function, one advantage of Chrome messaging API is that you can return message in the receiver side, so this way is more like calling a JavaScript function. We still start from the sender side:

{% codeblock lang:js event_handler.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/content_scripts/event_handler.js event_handler.js %}
/**
 * 
 * Open the caption editor for user when receive the open editor message from UI component.
 * It add a new iframe to current page and allow user to edit caption, make comments and do something else with the caption.
 * 
 * @author Junkai Yang
 * @param  {dom} frameDom
 * @param  {dom} videoDom
 */
var openEditor = function (frameDom, videoDom) {
    chrome.runtime.sendMessage({ application: "video_caption", type: "OPEN_EDITOR" }, function (response) {
        console.log(response.success);
    });
} 
{% endcodeblock %}

In the receiver side, the script `background.js` is: 

{% codeblock lang:js background.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/background/background.js background.js %}
chrome.runtime.onMessage.addListener(
    function (request, sender, sendResponse) {
        console.log(sender.tab ? "from a content script:" + sender.tab.url : "from the extension");
        if (request.application == "video_caption"){
            if(request.type == "OPEN_EDITOR"){
                chrome.tabs.executeScript(null, { file: "scripts/content_scripts/inject_editor.js", allFrames: true });
                sendResponse({ success: true });
            }
        }
    });
{% endcodeblock %}


## 4. Messaging between extension and native application
In our extension, we have an idea which allowing user voice control some functions in the captions editor such like making a bookmark. The idea comes up because we hope our user can still use the editor even when the video is in full-screen mode, naturally the shortcut key and voice command idea comes into our mind. 

To achieve this function, we have another guy who makes a native application in Java and we use this application to capture and analyze the voice. Thus, the use case is when user long press the CTRL key then we send a message to the voice command application and wait for the return message which contains the command string of user.

First, you will need to declare the permission you need in the `manifest.json`:
```js
{
    //......
    // Other configuration
    "permissions": [
        "activeTab",
        "https://ajax.googleapis.com/*",
        "https://api.coursera.org/*",
        "nativeMessaging"
    ],
    //...
}
```

Only background script can communicate with the native application. There is a good [example](https://chromium.googlesource.com/chromium/src/+/master/chrome/common/extensions/docs/examples/api/nativeMessaging) on Google developer website and most of my codes comes from there. 

However, I want a long session instead of one time communication, so I create a new background script `voice_command.js` and also change some code of the native application. For the native host, you need to configure something based on the operating system. It is really easy so please check the document [here](https://developer.chrome.com/extensions/nativeMessaging). But I also upload all of my native host code on Github, you can download and change if you want. [Github link](https://github.com/cyfloel0516/video_caption_chrome_extension).

Below is the code in the sender side:
{% codeblock lang:js voice_command.js  https://github.com/cyfloel0516/video_caption_chrome_extension/blob/master/scripts/background/voice_command.js voice_command.js %}
/**
 * This file is a backgroun script which is responsible for the handling of voice command.
 * It will receive the message from content script and redirect the message to a native application. Then it wait for 
 * the native application to return the voice command string.
 * It uses the Chrome native message passing ability to communication with native application.
 * 
 * @author Junkai Yang
 */
var port = null;

function onNativeMessage(message) {
    if (message.command == "ACTION") {
        console.log("Voice command native script receives: " + message.result);
        chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
            chrome.tabs.sendMessage(tabs[0].id, { application: 'video_caption', type: "VC_ACTION", action: message.result });
        });
    }
}

function onDisconnected() {
    chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
        chrome.tabs.sendMessage(tabs[0].id, { application: 'video_caption', type: "VC_INIT_FAILED" }, function (response) {
            //console.log(response.farewell);
        });
    });
    port = null;
}

function sendNativeMessage(text) {
    message = { "message": text };
    port.postMessage(message);
}

function connect() {
    var hostName = "com.jy.syr.chrome_native_message_passing"; // Change to you application
    port = chrome.runtime.connectNative(hostName);
    port.onMessage.addListener(onNativeMessage);
    port.onDisconnect.addListener(onDisconnected);
}

chrome.runtime.onMessage.addListener(
    function (request, sender, sendResponse) {
        connect();
        sendNativeMessage("SPEAKING");
    });
{% endcodeblock %}


In the receiver side, Google uses Python 2.7 in the example code and I change some code to make it works on Python 3. Also, I put all code in a while loop which make the long session possible. Moreover, this python script is actually a middle layer between the extension and the real Java application, so it means if you have your own application, you can easily change the python script.

{% codeblock lang:python VoiceCommand.py https://github.com/cyfloel0516/chrome_native_message_passing/blob/master/VoiceCommand.py VoiceCommand.py %}
# This file is an example to show the communication between chrome extension and python script.
# It also shows the message passing between python and java.
# This git repo is only the host part of example. Refer to the extension side on: 
import struct
import sys
import subprocess
import os
# On Windows, the default I/O mode is O_TEXT. Set this to O_BINARY
# to avoid unwanted modifications of the input/output streams.
if sys.platform == "win32":
  import os, msvcrt
  msvcrt.setmode(sys.stdin.fileno(), os.O_BINARY)
  msvcrt.setmode(sys.stdout.fileno(), os.O_BINARY)

# Helper function that sends a message to the webapp.
def send_message(message):
  # Write message size.
  sys.stdout.write( struct.pack('i', len(message)).decode('UTF-8') ) 
  # Write the message itself.
  sys.stdout.write(message)
  sys.stdout.flush()

# Thread that reads messages from the webapp.
def read_thread_func():
  message_number = 0
  while True:
    # Communicate with another application through subprocess module
    p = subprocess.Popen(["java", "-jar", os.path.dirname(os.path.realpath(__file__)) + "/MessageHandler.jar"], stdout=subprocess.PIPE)
    text = p.communicate()[0]
    send_message('{"echo": "%s"}' % text)

def Main():
#   p = subprocess.Popen(["java", "-jar", "./NativeMessageReceiver.jar"], stdout=subprocess.PIPE)
#   text = p.communicate()[0].decode()
#   sys.stdout.write(text)
#   sys.stdout.flush()
  read_thread_func()
  sys.exit(0)

if __name__ == '__main__':
  Main()

{% endcodeblock %}

## 4. Summary
So far, I have go through the basics and details of building a Chrome extension. I will write something about Gulp building for a React Chrome extension project in the next, also the last post of this topic. 

[Build a Chrome Extension with React and Gulp - Part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-1/)
[Build a Chrome Extension with React and Gulp - Part 2](/blogs/2016/06/27/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/)
[Build a Chrome Extension with React and Gulp - Part 3](/blogs/2016/06/27/Build-a-Chrome-Extension-with-React-and-Gulp-Part-3/)