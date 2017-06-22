---
layout: post
title: jQuery templates vs. MVC Partial Views
redirect_from:
- "/post/jquery-templates-vs-mvc-partial-views.aspx.html"
- "/post/jquery-templates-vs-mvc-partial-views.aspx.aspx/"
tags: [aspnetmvc, aspnet, mvc, templates, jquery]
---

### Introduction

Now that jQuery templates are becoming popular, many ASP.NET developers are using them on their sites. We know that both jQuery templates and partial views can achieve same result. There are even many blog posts that cover the details of jQuery templates, so I am not going to explain them here. Instead, I am going to show you when it is convenient to use jQuery templates and/ or partial views.

### Client-side templates vs. partial views

As you might know, jQuery templates can be thought as client-side view engine. Sometimes jQuery templates can complicate your code and may not bring you any benefit at all. They can be handy though if you know when to use them. Here are some guidelines that will help you decide:

 

*    Use partial views if data is not to be manipulated on the client. For example, if you are loading a list of items (eg books) and need to show them in table rows or whatever. Why would you need jQuery templtes?!
    Using jQuery templates will impose extra javascript code with no benefit.
*    Use jQuery templates if you are going to modify or refine the information returned from server before rendering to HTML. For example, if you have a list of items (eg books), you might need to modify their order or adjust time to browser’s local time before showing them, you will need to parse them on the client using some tool like jQuey templates.
*    Use jQuery templates if you need to show same information in more than one part of the DOM even if you are not planning to modify this information. For example, To render a list of books as table rows (`<tr>`) and as list items (`<li>`) at the same time, you will normally return a JSON containing an array of the books, and then parse it using two different views and show the lists in their corresponding parts in the DOM. However, partial views cannot do the same thing at the same time, it would require another round trip to the server which is not efficient.
*    Use jQuery templates when you want to pull data from 3rd party servers like twitter. For example, you can load tweets on the client using JSON format and then parse them using jQuery templates. You are unlikely to send this information (tweets) back to your server for the sake of rendering to HTML. It would be superfluous. However, if your server is acting as a proxy, a one that fetches tweets directly from twitter, it makes sense to use partial views.
*    Use jQuery templates if your site is build entirely using javascript and Web Services. This makes more sense in ASP.NET Web Forms (Classic ASP.NET), because ASP.NET Web Forms has no idea of partial views. So jQuery templates act as a view-engine on the client. It would an efficient approach, but very tedious!!
*    Use partial views when javascript is disabled on the client. Remember if javascript is disabled, jQuery templates won’t be available at all.
*    Use jQuery templates if you want to return different pieces of information from the server. For example, you can fetch a list of books, a list of authors, and a list of movies all at the same time in only one roundtrip to your server. You can do this using JSON format containing arrays for each one. With partial views, you need to perform multiple requests to fetch each array independently as ready HTML. Note you can also do this by returning direct javascript code that injects HTML elements into DOM. This approach is not recommended as it complicates your serverside code with javascript, which is likely to change over time.

### Summary

Recall that jQuery templates are much more flexible than partial views, but also more tedious. Sometimes they are not necessary and sometimes they are really handy. It is a trade-off between flexibility and complexibility. But you can also mingle between partial views and jQuery templates in your website.