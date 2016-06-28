---
title: Build a Chrome Extension with React and Gulp - Part 1
date: 2016-06-26 20:39:50

categories:
- Development

tags:
- JavaScript
- Gulp
- Chrome
- React

---
Currently, my team wants to move some function in our Captioning Project into a Chrome extension and then it can be used in many video website such like YouTube and MOOC such like Coursera. 

In our website, we just use JavaScript and jQuery to control our page and with the page complexity increasing, the coding and maintenance becomes more and more difficult and that's the incentive for me to use React in this Chrome extension. However, I won't talk too much about React and Gulp and I assume you already have basic knowledge of them.

I write this post to help myself get a better understanding and also hope to help someone in need. I believe this post can help you in building an extension but either you cannot find what you want or find something wrong, please refer to the [Chrome extension official document](https://developer.chrome.com/extensions/overview) or leave your comments.

From this post, I will show these basic things for building an extension:
1. Open a menu window in current page when user clicks the extension button
2. Manipulate the page with content scripts
3. Messaging between page component and extension script

*The repo of my extension can be found [here](https://github.com/cyfloel0516/video_caption_chrome_extension)*

<!-- more -->

## 1. Overview

At the very first, I want to briefly introduce my extension and then I will elaborate on each part of function in this extension.

The extension has a menu window and it shows when user clicks on the extension icon, then menu window shows on the top-right area on the current page like this:
![](menu_window.png)

Then, it has two buttons for user:
1. Load the captions found in our server to the video
2. Or open the captions editor window in current page
![](edit_window.png)

The extension includes functions:
* Get the video information and search captions from our server
* Load the sentence on the video with the video playing
* Allow user to correct caption, make bookmark, write comment and even voice control the editor

Actually, it is really a piece of cake for who are quite familar with web development and implment these functions in a traditional website. Though building an extension is very similar with that of buidling a website, there are still many differences and later I will go deeper and show you how to implement such an extension.

## 2. Basics
[Chrome Developer](https://developer.chrome.com/extensions/overview) page demonstrates some basics and key concepts for Chrome Extension development. However, after finishing the development of my extension, I would like to conclude the basics in a simpler way.

### Three layers of Chrome extension
Building a Chrome Extension is just like building a website but with some restriction. You need to declare the permission you need in manifest and you can only put specified function in specified layer. Once you know what function you need and where to place it, you can eaisly build your own extension.

#### **1. View Layer**
This layer includes the content you want to show to the user, such like the extension menu window and captions edit window below the YouTube player. 

**What you can't do in this layer:**
Taking DOM manipulation for example, if you put your component in an iframe, you can manipulate everything within this iframe but you can do nothing on the original website.  


#### **2. Content Script Layer**
Script in this layer runs in the same context of the web page which means you can do everything you want on the website. You can see the captions loading function in my extension and this function is done by a content script by change the DOM of YouTube page.

**What you can't do in this layer:**
Almost every function you need can be implemented in this layer but there still some restriction. First, you cannot listen to some event from the browser such like extension icon clicked event. Second, unlike the background layer which has the same lifecycle with browser, content script layer runs in the context of webpage. Last, you cannot communicate with local application in this layer, native application messaging function can be only implemented in the background layer. 


#### **3. Background Layer**
Background scripts are actived when the browser is running, so you can put your event handlers in this layer. For example, you should put the the icon clicked event handler here to open the menu window when user clicks the extension icon. Also, you can communicate with native application in this layer.

**What you can't do in this layer:**
You can't manipulate the DOM because script in this layer doesn't share the context with the page.

Knowing these layers can help you structure your project, you can take a look at the scripts folder in my repo [here](https://github.com/cyfloel0516/video_caption_chrome_extension):
```
|-- Scripts - All scripts
    |-- background - Place background scripts here
    |-- content_scripts - Place content sctipts here
    |-- components - Place react components here
    |-- video_api - scripts to perform some video manipulate works, used by content_scripts because only content scripts can access the DOM
    |-- *.app.js - View layer, render the react components and injected to page by content scripts when user clicks the icon

```

## 3. Summary
In part 1 I just briefly introduce some basics of Chrome extension, in the next few post I will show the details of buidling an extension.

[Build a Chrome Extension with React and Gulp - Part 1](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-1/)
[Build a Chrome Extension with React and Gulp - Part 2](/blogs/2016/06/26/Build-a-Chrome-Extension-with-React-and-Gulp-Part-2/)
