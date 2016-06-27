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

While I was developing the Captioning Project in Syracuse University, my team was trying to integrate our system with our main learning environment Blackboard. After doing some research and thanks to the great tool provided by [SPV Software](http://www.spvsoftwareproducts.com/), I eventually implemented the function to get the user and course data from blackboard. This post includes some information and steps which I think should be useful.

This post is for PHP and Blackboard but it should be similar to move to another language or learning system.

## Overview
LTI provides a standard way of integrating our application with learning management systems like Blackboard and Coursera. The most important data from learning management systems is of course the user and course. 

My code was written in PHP and only test with Blackboard but it should work on other learning tool system which provides standard LTI data.

<!-- more --> 

## How it works
Basically, the idea of LTI is sending a request to our server. This request contains some data, such like user and course, thus, what we need is to handle the request from a learning system and fetch what we want from the request. 

So the idea is to get the POST data from learning tool and save these data in PHP session object, then it redirect the request to a real page which needs these data. So based on my existing code, you just need to make some configuration then you can use the session object in your PHP page.

## Steps
1. You can download the LTI_Provider from [here](http://projects.oscelot.org/gf/project/php-basic-lti/frs/). My project uses PHP version but they also provide the version for Java and Windows appliaction. 

    The **LTI_Tool_Provider.php** file provides a base class. There are many events in this class so that you can override them with your own code. You can write your own handler class from scratch, but for simplicity, I use the demo code which is also provided by the author of LTI_Provide and make some changes to it. You can download the demo from [here](http://projects.oscelot.org/gf/project/php-basic-lti/frs/).

    You can access my demo code in my (github repository)(https://github.com/cyfloel0516/LTI_Demo/tree/master). I have put the LTI_Provider files and the implementation file together so that you can use them directly.

2. Execute the sql script **lti-tables-mysql.sql** to create tables for LTI_Provider. You should add these tables to your application database.

3. (Optional)Add your learning system information in LTI consumer table. 
    ```sql
    insert into lti_consumer(consumer_key, name, secret, consumer_name, comsumer_guid, enabled, created, updated) 
        values ("Syracuse University", "Syracuse University", "SecretKey", "Syracuse University", "05ede25ecb474fef8da9436faacdb6e4", 1, NOW(), NOW()) 
    ```
    *If you skip this step, the LTI_Provider will use the information provided by learning system. You should have your own way to generate the comsumer secret and guid.*

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
     



