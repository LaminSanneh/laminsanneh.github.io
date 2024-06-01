---
title: Html5 localStorage, dead simple
description: ""
date: 2013-04-20T09:16:52.388Z
preview: ""
draft: false
tags: []
categories:
    - Javascript
type: posts
slug: html5-localstorage-dead-simple
hero: /images/posts/local-storage.png
---

## Introduction

Lets get something straight, this stuff (Html5 localStorage) should be simple, dead simple, so here we go.

## Storing Simple Data (e.g. numbers and strings)

localStorage.setItem(keyName, myString);

Notice the capital letter “S“ in localStorage , its necessary.

Don’t worry localStorage is a global variable in your browser, so use it freely from anywhere.

## Retrieving Simple Data

localStorage.getItem(keyName);

## Storing Object

localStorage.setItem(keyName, JSON.stringify(yourObject));

When storing objects, you would want to serialize your object into a string using “JSON.stringify(yourObject)” because local storage only stores simple data (i.e. numbers and strings), so you object gets converted into a json string.

## Retrieving Object

JSON.parse(localStorage.getItem(keyName));

When retrieving objects, you would want to deserialize its string form from your storage back into an object using “JSON.parse” method.