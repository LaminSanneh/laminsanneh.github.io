---
title: Underscore js, the missing Linq for c-sharp developers
description: ""
date: 2013-05-22T12:50:17.194Z
preview: ""
draft: false
tags: []
categories:
    - Javascript
type: posts
hero: /images/posts/underscore_js.png
---

## Introduction

We all have our favourite languages that we feel most comfortable with for various reasons. One such language for me is the c-sharp language. One of the primary reasons why I love the language is a really cool feature called Linq.

I simply cannot emphasize the amount of time this feature has saved me while doing work. It integrates seamlessly into my workflow and find myself almost subconsciously using it. And did I say super awesome chaining you could do as well just like in jquery.

Talking of jquery, as of late, most of my work has been involved with javascript and json and such stuff. I find myself missing this really cool linq feature. I had to write a lot of my own custom extension methods to manipulate arrays and json objects. If you are in the same boat as me, you might find the following examples very useful. Some examples of linq features you can find substitutes for in javascript are below.

The underscore “_” is a global variable which is available to you after including the underscore.js script in you application.

If you have a javascript array for example, myArray, you can do the following operations on it using underscore.
## 1. Looping through array elements

_.each(myArray, functionToRun)

where the custom function “functionToRun” is a function which you have written an implementation for and it is passed a single item in the array during each iteration

## 2. Filter array elemnts based on a “where” criteria( familiar, linq people?)

_.where(myArray, properties)

In this case, the variable properties is a key value object which is used to filter the array “myArray”

For example if you have an array in the format :

var myArray = [{
        name : "Bob"
    },
    {
        name : "Alex"
    },
    {
        name : "Bob"
    }
]

passing the array above with the following filter property :

var properties = {
   name : "Alex"
}

would filter through the array and return a single item which matches the criteria specified in the “properties” variable. In this case the value returned is an array with the object having a name of “Alex” as shown below :

[{
   name : "Alex"
}]

## 3. Get first item from an array

Using the following method passing your array as the first argument would return you the first item of the array

_.first(myArray)

## 4. Taking the first n items from an array

Using the same method above but this time passing an optional number as a second argument will return you the first n items in the array

_.first(array, n)

My C-sharp buddies are probably thinking of the Take method right now.

While c-sharp developers coming from the .net world will be familiar with the examples above, it will also be useful for those who were primarily developing in javascript and need some extra firepower with object manipulation.

If you need reference and want to find some more useful features like the one I just showcased above, you can head over to the official Underscore Js website at http://underscorejs.org/

That’s it for now. Hopefully this would serve you well for the many awesome JavaScript years ahead of you. Thanks for reading and please do let us know if you have anything to add in the comments below.
