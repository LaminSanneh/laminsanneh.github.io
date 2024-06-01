---
title: Understanding the ASP.NET MVC 4 Project Structure and Folders
description: ""
date: 2013-02-10T09:24:20.684Z
preview: ""
draft: false
tags: []
categories:
    - ASP .Net
type: posts
slug: understanding-asp-net-mvc-4-project-structure-folders
hero: /images/posts/dot-net.png
---

## Introduction
When I first started doing Net Mvc programming, one of the first stumbling block I had was understanding the Asp.Net MVC 4 project structure and folders. So I decided to share some of what I know now that I’m comfortable with them. This tutorial applies to both Visual Studio 2010 and Visual Studio 2012, sorry Visual studio 2008 and Visual studio 2005 folks, I was not programming .Net and or C# back in those days. This is not a What is Visual Studio tutorial, but I have a separate post coming up soon for that if you’re new to it as well. The language does not matter, either C# or Visual Basic will do. We’ll use a new Asp.Net MVC3/4 web application as the example for this post.

##  Create a New Asp.Net Mvc 4 solution
Start by creating a new project by going to file new project on the upper left of your Editor.   Using VS 2012 with the 2010 blue theme applied

##  Solution ‘MvcApplication3’
This is the name of your solution. A solution is like a container for your whole application. In .Net world, it is the global container which contains other projects. A single application can have only one solution but an Mvc 4 solution can contain multiple projects. An example of a project is described below after the image showing a newly created solution.   Visual studio project structure fresh

##  MvcApplication3 – A Project Name
This is the name for a single project. It is contained in the solution we just described above. In this project we have direct children folders and or files. The children we would now briefly describe below.

## Properties
This is where we have information related to our project which helps in uniquely identifying it. It is sort of like a meta file for our project.

## References
In here we have references to other projects in our solution and DLL dependencies which we our project needs to run.

## App_Data
Inside of here is where you can store data related to your website. Most of the time you people use it to store portable file-based databases like an .mdf file for example.

## App_Start
The app start is somewhat of a special folder in that by default it contains various configuration files. Below are the default files in it upon creating a new project.   App Start Folder

## Content
This is where you would put stylesheets and optionally images for your website.

## Controllers
Your application controllers live in here. Just Briefly, a controller is like a decision maker for your application or project. Any requests made by an application user gets directed to one of several controller which then decides what to do next, be it fetch data or store it in the database e.t.c.

## Models
Your application models reside in here. For example if your application were to represent entities like Person, Car e.t.c., you’d store those classes here.

## Scripts
In here as I’m sure you guessed it, is where you organise your  javascript files.

## Views
The view files for you project reside in here. If you’re wondering what a view file is. It is a file containing html and a little bit of c# code for displaying your pages. C# view files end with .cshtml while VB.Net view files end with .vbhtml . Views Folder

## Global.asax file
In here, you have project wide settings registration. Any configurations you have made in you application are registered or de-registered in this file. For e.g. routes.

## Packages.config file
This file is where your referenced packages in references are described. For e.g. version numbers of referenced packages along with their .net targets are detailed in this file.

## Web.config
Inside of the web.config file is where you store other setting which you need to change without recompiling your code. It is an xml file so does not need compilation. You can also store information like connection strings for your databases, membership providers for website authentication e.t.c.

## Web config file

Sneek peek at part of an asp.net mvc 4 web.config file


## You still have choices
Just Remember, asp.net folder structure is mostly for best practice as most of the structure can be over-ridden in favour of your choice of structure. But most of the time I find that I need to only add extra folders but leave the pre existing default one intact as they pretty much do the job that I want in terms of structuring my code. So there we have it,  I hope that helps clear up some confusion. Please comment below if this was of any help.
