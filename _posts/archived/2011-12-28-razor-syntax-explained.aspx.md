---
layout: post
title: ASP.NET Razor Syntax Explained
redirect_from:
- "/post/asnet-mvc-razor-syntax-explained.aspx.html"
- "/post/asnet-mvc-razor-syntax-explained.aspx/"
tags: [aspnet, razor, syntax]
---

### Introduction
In this blog post, I am going to talk about Razor syntax that you might find useful. If you already know Razor well, this will be a refresher.

Razor syntax is simple and easy to understand. It requires a minimum amount of keystrokes, unlike WebForms. The `@` character marks the beginning of code. This code can be an expression or a code block.

An expression does not end with a semicolon, and it returns a value after its evaluation. For example,

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-1.png)

Each of the expressions in the previous lines returns some value. We didn’t need to add a semicolon at the end. Razor is smart enough to know where the expressions ended, so we didn’t have to close the @ tag because it ended with an html markup. However, sometimes Razor cannot guess where the expression ends, and in this case we need to use parenthesis.

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-2.png)

Code blocks can be defined using `@{}`. Code blocks do not return any value. They just contain some code that will be executed. It is useful for converting or declaring variables that will be used in the view.

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-3.png)

Sometimes we want to set a different color for alternating rows in a table. Below is an example:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-4.png)

If the row is even, we give it a different background color that is defined in the CSS class 'highlighted'.

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-5.png)


It is worth saying that you cannot define a function inside a block, but you can define helpers and delegates.

Compare the following code snippets. The last one won’t compile:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-6.png)

`Enumerable.Range` generates a sequence of integers for you. We are storing the numbers from 1 to 10 in an array. In code blocks, Razor expects to find code unless we tell it this is text. However, sometimes like in the first foreach block, Razor can recognize that <p> is an html markup, so it compiles fine. In the last foreach code block, Razor cannot guess that "current.." is some text, so we need to tell Razor this is a text and not code. We have a few options:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-7.png)

The `<text></text>` block tells Razor this is markup, not code. Likewise, `@:` also is followed by markup; however, you cannot use multiple lines in it like in <text>.

By default, Razor html encodes all strings. For example,

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-8.png)

Outputs the following:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-9.png)

We can use Html.Raw for this purpose:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-10.png)

Which does what we need:

**Hello Razor!**

Comments in Razor can be inserted using @* *@. Compare the following:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-11.png)

The difference between a Razor and HTML comment is that the Razor comments are never sent to the client, just like (server-side code) C# comments. If you view the source of the rendered HTML document, you’ll only see the HTML comment:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-12.png)

As you can see, the C# and Razor comments are never sent to the client. Server side comments are intended for development purpose only.

`ViewBag` is a new dynamic property that gives us a more convenient syntax than `ViewData`. In fact, ViewBag is a wrapper for `ViewData`. To prove this:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-13.png)


### Razor Declarative Helpers

It is often necessary to generate the same HTML markup to be used in different places in views. Repeating the same HTML code is not efficient. If you need to make a small change, you have to modify all the repeated blocks of HTML which is tedious and prone to error. HTML helpers allows to us to overcome this problem by using the same HTML markup.

There are two ways to reuse HTML markup: Razor declarative helpers and extension methods. I'll cover extension methods in a different post.

If you are going to use a helper in a single view, it makes sense to define it in that view file. Inline helpers are declared using the keyword `@helper`

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-14.png)

Which renders the following

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-15.png)

If you plan to reuse this HTML code in a different view, you can move this helper definition to a global file in App_Code folder. Add a new folder and call it "App_Code". In this folder, add a new Razor view and give an appropriate name such as MyHelpers.cshtml

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-16.png)

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-17.png)

Now you call the helper method with the view file name like below:

![](/images/posts/archived/asnet-mvc-razor-syntax-explained-18.png)

### Declarative Helpers Limitations

Declarative helpers are sweet. However, I noticed there are some limitations. If you define your helpers in App_Code, you won’t be able to use Html helpers like Html.Encode or Html.Label. Nor can you use Url.Content or Ajax helpers.

I hope this helps.
