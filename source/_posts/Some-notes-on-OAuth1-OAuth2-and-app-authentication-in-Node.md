title: Some notes on OAuth1/OAuth2 and the implementation in Node.js
date: 2017-01-22 21:58:01

categories:
- Development

tags:
- JavaScript
- Node.js
- Authentication
---

Authentication is one of most important things for almost every application today. A traditional way to do this is to validate username/passport combination but there is an easier and more popular way to authenticate the user: connect your app(either web or mobile) to Google or Facebook and you can get the user information from Google or Facebook easily.

I believe it is not hard to integrate an app with Google or Facebook, all you need is to simply follow the docs step by step. But I also believe it will be better if we can get a fully understanding of how it works. That's why I decided to write some notes based on my experience on this topic.

<!-- more -->

## Basic idea of OAuth1/OAuth2

Now Google and Facebook are both moving to OAuth2 but the basic idea behind OAuth1 and OAuth2 is very similar: user information is sensitive data so all the requests require authentication. The key steps for OAuth1 and OAuth2 can be concluded as:

1. App redirects user to Google/Facebook login page for authorization
2. User grants the authorization
3. Google/Facebook redirects user back to app with a temporary `authorization code`
4. App exchanges the `authorization code` with server for `access token` 
5. App uses the `access token` to request the sensitive data from server

So you can think of it in this way: app wants to access user's data but the data is in an repository. So the user go to the repository(Google/Facebook) and sign a authorization sheet(Authorization code). Then the user gives the sheet to the app says that "Hi app, you can go to repository and get all you need(Access token) with this sheet, I have signed it."

Here comes the question, what is the difference and how the difference makes OAuth2 more popular today? 

### Difference between OAuth1 and OAuth2
The short answer is OAuth2 puts a lot of things on SSL connection while OAuth1 requires some more steps to secure the communication between client and server. So simply put, OAuth2 is simpler than OAuth1 because SSL can help with some security problem.

Basically, the steps 1-5 above are all required and almost identical in both OAuth1 and OAuth2. However, because OAuth1 doesn't rely on SSL, both app and server need to hold a `shared secret`. This `shared secret` will be used to sign the request and the server will verify the signature with the same `shared secret`.

So it looks like that validating signature in OAuth1 is not that difficult but why people keep moving to OAuth2? Here are two important reasons in my opinion:

1. Signing the request string in OAuth1 is difficult and the behavior is heavily depends on the language, library or server implementation. Generating the signature is almost the most painful part in OAuth1 for many people.

2. The request of signature in OAuth1 makes it inappropriate for non-browser application, for example, the TV-application. The reason in these device you may have difficulty opening the login page and the restriction makes you impossible to delegate the login process to another device, for example the user's smart phone because only the device has the `shared secret`.

In short, OAuth2 removes the requirement of signature and make things easier. However, the former author of OAuth2 wrote a blog [OAuth 2.0 and the Road to Hell](https://hueniverse.com/2012/07/26/oauth-2-0-and-the-road-to-hell/) to express his concern on the security problem of OAuth2. But as far as I can see, OAuth2 becomes more and more popular today so I don't see any reason to use OAuth1 as long as you trust the security provided by SSL.

### Some notes for myself
Here are some important notes for myself on some important questions of OAuth1 and OAuth2:

1. What is the most significant difference between OAuth1 and OAuth2?
    
    As mentioned before, OAuth2 relies on SSL connection to make communication safe while OAuth1 requires client and server to use the `shared secret` to generate and verify the signature. So OAuth2 is simpler becuase we don't need to deal with the signature anymore.

2. What are the benefits from using OAuth2:

    OAuth2 is simpler and friendly to devices like Google-TV or other non-browser based platform. Also, you don't need to care about the signature anymore. 

3. In OAuth1/OAuth2, why do we need to exchange the temporary authorization code for access token. Why cannot the server send it to the callback url:
    
    One reason is that the server needs to know who is requesting the access token. So only the request from our application will contain the "application information" and only in this way that the server can tell who is going to use the access token.
    Another reason is that OAuth2 introduce "refresh token". The exchange step allows app to obtain a new valid access token.


## Authentication in Node.js
Authentication in a Node.js server application is not difficult and here what I mean by Node.js server is Express application. Of course you can write your own codes to connect your Express app to Google or Facebook. But a better way is to use existing library like [Passport](http://passportjs.org/). It provides a uniform way to write authentication code for different authentication provider like Google or Facebook and it can definitely make your code clear and robust.

Passport uses different strategies for different authentication providers, but the interface for your application are the same. You just need to apply Passport as a middleware in your Express application and implement the authentication interface. For implementation detail please refer to Passport official site, following is only some breif notes on how to implemente authentication in an app based on my experience.

## Two level authentication
Actually, there are typically two levels of authentication in an application. First level is to authenticate the user with Google or Facebook. The second level is the authentication for the application's own API. To do this in my application, I issue another token to client for future API access instead of using the token which comes from Google or Facebook. 

I use [JWT](https://jwt.io/) in my application. JWT is a self-described token so it contains the information of issuer, expire time and access scope. So after user login with Google or Facebook, the server will issue a new JWT access token and return it to client. 

## Authentication for web application
Authentication for web is quite simple because everything is in the browser and everything is about JS. So what you is to pick a Passport strategy, fill in your own code and enjoy it! For more detail you can check the official docs of [Passport](http://passportjs.org/).

## Authentication for mobile device
The more difficult part is the authentication for mobile device. The steps for this kind of authentication will be some like:

1. Mobile app register a particular URL schema like "my-app://..."
2. Mobile app the Google/Facebook login page with the callback url "my-app://..."
3. After user grants the access, the browser will redirect to the callback and OS will handle the URL and redirect you back to the app. Then the app can exchange the authorization code for access token.

There are existing login library from Google or Facebook so you can use them directly. Basically, the library is just providing a button to open the login page, register a particular URL schema and store the access token somewhere for future use. As long as you understand the OAuth workflow you can easily understand how these libraries work.

But you should notice that we only have access token in mobile device so far and our server don't know anything about the user. So similar to what we do in an web app, we need to send the access token back to our server through HTTPs connection and server can use this token to get information from Google or Facebook and also issue the JWT access token to mobile client.

## Summary
To sum up, OAuth2 and OAuth1 share the same idea but OAuth2 relies on SSL so it is simpler than OAuth1 to some extent. When you are developing a server application with Node.js and want to allow your user sign in with Google or Facebook, you can use your Passport and issue your own JWT access token for your API access control.

And for implementation detail you can just search the keywords "JWT, Passport js, OAuth2" on Google and there will be hundreds of articles tell you how to do it.


