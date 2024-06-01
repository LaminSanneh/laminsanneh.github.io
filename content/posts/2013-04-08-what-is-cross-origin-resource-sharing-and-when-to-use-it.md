---
title: What is Cross Origin Resource Sharing and when to use it?
description: ""
date: 2013-04-08T09:32:53.887Z
preview: ""
draft: false
tags: []
categories:
    - Javascript
type: posts
slug: cross-origin-resource-sharing
hero: /images/posts/cors.png
---

## The Problem

When developing highly ajax enabled applications using javascript, you may not always be fortunate to have both the client and server side of the application be hosted on the same domain. You might be asking, what does this mean? Good question.

I am confident that you know what a domain is. If not, then have a quick look here at the wikipedia article. Ok, so lets say you have a domain named http://www.mydomain.com.

And you have an application hosted on that domain. By default that application would be using port 80 of the domain like so : http://www.mydomain.com:80 .

If you happen to have another (server side) part of the application hosted on another domain like http://www.myotherdomain.com . When your client side part of the application tries to connect to the server side using any of the four CRUD ( create, read, update and delete) through ajax.

Most modern browsers, because of security reasons, would not let that operation happen. You would be greeted by a Cross Origin Resource Sharing error if you try to do such a connetion.

What this error means is, the server would not let your client side application access any resource in server side application. Now you may be clever and be like “hey!”, what if I put both client and server side on the same domain?

Yes, pretty clever but unless both client and server side share the same port and are on the same domain. The application considers the two sides to be on different domains (i.e. considers http://www.mydomain.com:80 and http://www.mydomain.com:90 to be two different domains) and would still throw the CORS error we encountered earlier on.

Dont be alarmed too much though because there is a solution which I am about to share with you. This is one trick I discovered while working on my chat application( which is still in progress ) whose client and server side are hosted in a similar manner (on the same domain but on different ports).

## There are two steps involved in this solution

1. You would need to make you client side add an http header to it request so as to let the server know from what domain( including the port) the request is originating from. This is done adding a header with a key value pair of ORIGIN and domainname respectively as so :

Origin: http://www.example-client.com: 68

assuming port 68 is the port on which your client app is hosted on

2. Now you have to make a change in you server side application so it allows requestes from foreign domains. How you do that is to add an http header with a key value pair of “Access-Control-Allow-Origin” and “domain-to-be-allowed-to-make-requests” respectively to any response it sends for any foreign domain requests, like so :

Access-Control-Allow-Origin: http://www.example-client.com:100

This way when the server receives a request with the ORIGIN header. If that matches any of the allowed domains in it response, then the two applications are able to communicate.

Whew!, that was a tough one but hopefully now that won’t be an issue for you any more. That’s it for now and as usual, please let us know in the comments if you have anything else to add or if you have any questions. Thanks for reading.
