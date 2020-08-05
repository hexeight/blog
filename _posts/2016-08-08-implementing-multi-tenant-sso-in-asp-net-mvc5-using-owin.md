---
id: 581
title: Implementing Multi-tenant SSO in ASP.net MVC5 using Owin
date: 2016-08-08T23:47:59+00:00
author: Omkar
layout: post
guid: http://omkarslab.com/?p=581
permalink: /2016/08/08/implementing-multi-tenant-sso-in-asp-net-mvc5-using-owin/
sfw_pwd:
  - IQ9c9sWI0Jgc
categories:
  - General
---
Decorating your SaaS offering with multi-tenant Single-Sign-On (SSO) can be a huge benefit, allowing your enterprise users to switch between their Office 365 home and your web app seamlessly.

While you can find a few resources that help you build just that, most of them are in the form of a project template or a complete solution on Github. Like this one ([TodoListWebApp](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-multitenant-openidconnect/tree/master/TodoListWebApp)). For my projects I prefer to keep them clean, and write as much code by myself; avoiding templates that carry more capabilities than I need, or even carry out operations that I don’t completely understand.

You can find my complete tutorial here <https://blog.puneusergroup.org/building-a-multi-tenant-active-directory-web-app/>