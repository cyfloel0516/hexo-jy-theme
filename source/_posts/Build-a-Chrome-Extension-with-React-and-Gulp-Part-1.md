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
Currently, my team wants to move some function in our Captioning Project into a Chrome Extension and then it can be used in many video website such like YouTube and MOOC such like Coursera. 

In our website, we just use JavaScript and jQuery to control our page and with the page complexity increasing, the coding and maintenance becomes more and more difficult. And that's the incentive for me to use React in this Chrome Extension.

I write this post to help myself get a better understanding and also hope to help someone in need. I believe this post can help you in building a extension but either you cannot find what you want or find something wrong, please refer to the [Chrome extension official document](https://developer.chrome.com/extensions/overview) here or leave your comments.

From this post, I will show these basic things for building an extension:
1. Open a menu window in current page when user clicks the extension button
2. Manipulate the page with content scripts
3. Messaging between page component and extension script

Meanwhile, I will show how to use React and Gulp to make development easiler and better.

<!-- more -->

## 1. Overview

At the very first, I want to briefly introduce my extension and then I will elaborate on each part of function in this extension.

The extension has a menu window and it shows when user clicks on the extension icon, then menu window shows on the top-right area on the current page like this:
![](menu_window.png)

Then, it has two buttons for user:
1. Load the captions found in our server to the video
2. Or open the caption editor window in current page
![](edit_window.png)

The extension includes functions:
* Get the video information and search captions from our server
* Load the sentence on the video with the video playing
* Allow user to correct caption, make bookmark, write comment and even voice control the editor

Actually, it is really a piece of cake for who are quite familar with web development and implment these functions in a traditional website. Though building an extension is very similar that of buidling a website, there are still many differences and next I will go deeper and show you how to implement such an extension.

## 2. Basics
[Chrome Developer](https://developer.chrome.com/extensions/overview) page demonstrates some basics and key concepts for Chrome Extension development. However, after finishing the development of my own project, I would like to conclude the development in my own way.

### Three layers of Chrome extension
Building a Chrome Extension is just like building a website but with some restriction. You need to declare the permission you need in manifest and also you can only put some specified function in some layer. Once you know what function you need and where to place it, you can eaisly build your own extension.

#### 1. View Layer
This layer includes the content you want to show to the user, such like the extension menu and the 
#### 2. Content Script Layer

#### 3. Background Layer


## Write a menu window for your extension


