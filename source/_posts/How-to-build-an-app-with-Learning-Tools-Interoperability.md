---
title: How to build an app with Learning Tools Interoperability
date: 2016-06-22 19:01:26

comments: true

tags:
- LTI
- Web
- PHP

categories:
- Development
---

While I was developing Captioning Project in Syracuse University, my team was trying to integrate our system with our main learning environment Blackboard. After doing some research and thanks to the great tools provided by [SPV Software](http://www.spvsoftwareproducts.com/), I finally implemented the function which gets the user and course data from blackboard. This post includes some information which I think is important and useful.

This post is for PHP and Blackboard but the development in other language or configuration in other learning system should be similar. 

## Overview
LTI provides a standard way of integrating our application with learning management systems like Blackboard and Coursera. The most important data from learning management systems is of course the user and course. 

My code was written in PHP and only test in Blackboard but it should work on other learning tool system which provides standard LTI data.

<!-- more --> 

## How it works
Basically, the idea of LTI is sending a request to our server. This request contains some data, such like user and course. Thus, what we need is to handle the request from a learning system. 

So the idea is to get the POST data from learning tool and save these data in php session object, then it redirect the request to a real page which needs these data. So all you need to do is to make some configuration then you can use the session object in your php page.

## Steps
1. You can download the LTI_Provider from [here](http://projects.oscelot.org/gf/project/php-basic-lti/frs/). My project uses PHP version but they also provide the version for Java and Windows appliaction. 

    The **LTI_Tool_Provider.php** file provides a base class. There are many events in this class so that you can override them with your own code. You can write our own handler class from scratch, but for simplicity, I use the demo code which is provided by the author of LTI_Provide and make some changes to it. You can download the demo from [here](http://projects.oscelot.org/gf/project/php-basic-lti/frs/).

    You can access my demo code in my (github repository)(https://github.com/cyfloel0516/LTI_Demo/tree/master). I have put the LTI_Provider files and the implementation files together so that you can use them directly.


2. Execute the sql script **lti-tables-mysql.sql** to create tables for LTI_Provider. You should add these tables to your application database.

3. (Optional)Add your learning system information in LTI consumer table. 
    ```sql
    insert into lti_consumer(consumer_key, name, secret, consumer_name, comsumer_guid, enabled, created, updated) 
        values ("Syracuse University", "Syracuse University", "SecretKey", "Syracuse University", "05ede25ecb474fef8da9436faacdb6e4", 1, NOW(), NOW()) 
    ```
    *If you skip this step, the LTI_Provider will use the information provided by learning system.*

4. Configure the provider information in Blackboard, just fill the form with the value of the consumer record you added to the database.
    ![](blackboard_configuration.png)
    **Remember, you should point the url configuration in blackboard to your **connect.php** file instead of your application url!**

5. Now you can redirect the request to your own page by editing the **redirect.js** file.


## Data format in LTI
You can see all available data from learning system in the *onLaunch* function**connect.php** file. All of these data are already set to SESSION object so you can use them in your own page.

## Other information
* See my demo code in this [repository](https://github.com/cyfloel0516/LTI_Demo/tree/master)
* You can check the lti configuration steps for blackboard: [LTI in Blackboard](http://library.blackboard.com/ref/df5b20ed-ce8d-4428-a595-a0091b23dda3/Content/_admin_app_system/admin_app_basic_lti_tool_providers.htm)
* I also write a document about how to configure LTI tool in blackboard: [how-to-setup-bb-environment.pdf](how-to-setup-bb-environment.pdf)
     



