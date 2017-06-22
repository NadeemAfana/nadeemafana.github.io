---
layout: post
title: Session Tracker plugin
redirect_from:
- "/post/session-tracker.aspx.html"
- "/post/session-tracker.aspx/"
tags: [aspnetmvc, aspnet, mvc, session]
---
**[Download Code](https://github.com/NadeemAfana/Session-Tracker-plugin)**

### Introduction

When you log on to a web site using your credentials, normally the server issues a ticket to your web browser, from that moment on, every time you access a protected page, this ticket is sent to the server so you won’t have to type your username and password again. This ticket is commonly stored in a cookie in your browser’s memory, and it ends when close your browser’s window. Additionally, the cookie has expiration date and time associated with it. When you make a request to the server, it checks the ticket’s date and time, if current date and time are later, it’s said that your session has expired, so you’ll need to log on again. If the ticket is still active, the server may decide to renew your ticket by adjusting its date and time. The algorithm it uses to renew a ticket depends on the platform you are using. Although the examples of this post are for ASP.NET, this plugin is not tied to ASP.NET or any technology. You can use it in any web technology that you like.

### The problem

When the session ends inadvertently, the user may not be able to complete a request. For example, let's say a user is filling in a form with long text fields, if session expires before they finish filling out the form and the user clicks on “submit”, the request will be rejected and all the work might be lost! This jQuery plugin will display a nice dialog box to alert the user about their session, and allows them to renew it if they want to.

![](/images/posts/archived/session-tracker-1.png)

Another good use is when your page is using asynchronous (AJAX) requests. Let's say a user is reading page 1 of some text on your web site and their session has expired, now they want to load page 2 using AJAX. In this case, ASP.NET will re-direct request to your web site's log on page. You won't like to see your log on page being displayed instead of your real text.

### Ticket Overview in ASP.NET

In ASP.NET, there are two ways to set ticket's expiration:

1.    Sliding expiration: It resets the expiration time for a valid authentication cookie if a request is made and more than half of the timeout interval has elapsed. If the cookie expires, the user must re-authenticate. if the ticket’s half life has elapsed, the server will renew the ticket.
2.    Absolute expiration: The cookie will always expire after a specified amount of time.

Ticket's expiration parameters can be configured in web.config file:

```xml
<authentication mode="Forms">
  <forms loginUrl="~/Account/login.aspx"
    timeout="20"    
    slidingExpiration="false" />
</authentication>
```

If slidingExpiration is set to true, sliding expiration mode is used. If it's set to false, absolute expiration takes place. The timeout parameter specifies the number of minutes before the cookie expires.

So as you might guess, this tool is not suitable for absolute expiration as there will be no way to extend the ticket's lifetime other than manual authentication.

**NOTE** There is common confusion between session state and browsing session. Session state is used to store user-specific data (e.g. user’s name, ID, items) that needs to persist across web page requests. Those items are discarded when the user is not accessing the site for a while. Browsing session refers to the authentication ticket issued to the user to allow him or her to navigate through protected pages in the web site. Session state and authentication ticket are stored in two different cookies, ASP.NET_SessionId and .ASPXAUTH, respectively. It is important to remember that authentication cookie (ASPXAUTH) is issued only to logged on (authenticated) users whereas Session state cookie (ASP.NET_SessionId) can be issued to both logged on (authenticated) and public users.


### How To Use It

It is very simple, all you need to do is add the following lines of code to your master page, so it spans across all your site pages. You may need to js files path.

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
&lt;head runat=&#34;server&#34;&gt;&#10;   &lt;script src=&#34;../../Scripts/jquery-1.4.1.min.js&#34; type=&#34;text/javascript&#34;&gt;&lt;/script&gt;&#10;    &#10;    &lt;% if (Request.IsAuthenticated){ %&gt;&#10;        &lt;script src=&#34;../../Scripts/jquery.nad.session.js&#34; type=&#34;text/javascript&#34;&gt;&lt;/script&gt;&#10;        &lt;script type=&#34;text/javascript&#34;&gt;&#10;            $(function () {&#10;                $.sessionTrack(20 * 60, {&#10;                expireAfter : &lt;%= (((System.Web.Security.FormsIdentity)Page.User.Identity).Ticket.Expiration - DateTime.Now).TotalSeconds %&gt;&#10;                });&#10;            });&#10;            &#10;        &lt;/script&gt;&#10;    &lt;%} %&gt;&#10;    &#10;&lt;/head&gt;
</pre>

Here is the basic call to the plugin:

```javascript
$.sessionTrack (timeout, settings);
```

The first parameter timeout is the timeout which appears in your web.config file but in seconds instead of minutes, so it is multiplied by 60. So 20 minutes = 20 * 60 seconds. The second parameter specifies many options to control the behavior of the dialog box. The most important one of them is expireAfter which specifies the real number of seconds before the ticket expires. The difference between timeout and expireAfter is that expireAfter will always be less than or equal to timeout because the ticket's life decreases over time. This value can be obtained by reading the difference between the ticket's expiration time and now:

```csharp
(((System.Web.Security.FormsIdentity)Page.User.Identity).Ticket.Expiration - DateTime.Now).TotalSeconds
```

![](/images/posts/archived/session-tracker-2.png)

If 2 minutes is very short or long, you can modify it easily:

```javascript
$.sessionTrack(20 * 60, {
   showBefore: 4 * 60 // show dialog 4 minutes before expiration
   });
```

Clicking on renew will renew the ticket, whereas clicking on ignore will hide the dialog and when session expires, the current page will be reloaded so the user gets re-directed to the logon page. This behavior hides protected content when session expires. It is useful on public computers where more than one person uses the same computer.

If you would like to re-direct user to a specific page instead of refreshing the current page:

```javascript
$.sessionTrack(20 * 60, {
      redirectUrl: "http://mysite.com/session-expired.aspx" 
   });
```

This page can show a friendly text like "Your session has expired" along with authentication form, which will enhance user experience.

When the user clicks on renew button, the current page is reloaded using AJAX in order to renew the ticket. If the current page is expensive to load in terms of performance, you could tell session tracker plugin to load an explicit page instead:

```javascript
$.sessionTrack(20 * 60, {
    callbackUrl: "http://mysite.com/blank-page.aspx" 
   });
```

This page can be empty for faster loading since the whole idea is just to make a request to the server.

If you do not want that page to be loaded using AJAX for some reason, hidden iframes can be used instead:

```javascript
$.sessionTrack(20 * 60, {
   callbackUrl: "http://mysite.com/blank-page.aspx",
   useIframes: true // use hidden iframe instead. 
   });
```

You can also change the default text that appears on dialog box. Here is the Spanish version of the text:

```javascript
$.sessionTrack(20 * 60, {
            captionText: "Su sesión expirará pronto!",
            renewText: "renovar",
            ignoreText: "ignorar"
   });
```

![](/images/posts/archived/session-tracker-3.png)

If you notice the dialog box also uses a modal overlay panel to block the page when it appears, you can disable this feature like this:

```javascript
$.sessionTrack(20 * 60, {
            useBlock: false // do not block page when dialog appears
   });
```

### Understanding Cookie Reset Time

As mentioned above, ASP.NET resets the expiration time for a valid authentication cookie if a request is made and more than half of the timeout interval has elapsed. However, for other web technologies, this value might be different. It can be changed easily:

```javascript
$.sessionTrack(20 * 60, {
    resetAfter: 10*60 // The number of seconds elapsed before ticket’s expiration time is reset. In ASP.NET, it's 1/2 of timeout.
   });
```

he code above resets the expiration time when 10 minutes of the ticket’s lifetime have elapsed. You won’t have to change this value for ASP.NET.

### Perform Custom Action Before/After Expiration

You may need to perform some custom action when session is about to expire:

```javascript
$.sessionTrack(20 * 60, {
  beforeExpire: function(){
    // This code runs before dialog box appears
},

  afterExpire: function(){
    // This code runs after session has expired.
    // return false; --uncomment to prevent redirection/refresh.
}

   });
```

You need to remember that `beforeExpire` does not necessarily mean session will expire, if the user clicks on "renew", their session will be renewed.

### Customizing Dialog Box

If default appearance of the dialog box does not comply with your site design, or you simply do not like it, it can be easily customized. You will need to create a panel using HTML and pass its ID to the panelId parameter:

```javascript
$.sessionTrack(20 * 60, {
      panelId: "my_dialog_ID"
   });
```

However, you need to consider the following:

1.    The "renew" button can be any element that supports "click" event like `<a>`, `<div>`, `<input>` and its ID must be "session-renew".
2.    The "ignore" button can be any element that supports "click" event and its ID must be "session-ignore".
3.    The modal overlay panel is optional but if used, its ID must be "session-block". If you wish not to use it, set useBlock parameter to false.
4.    You should implement all styles to your customized panel. You can add the styles to you style sheet and use images.
5.    The captionText, renewText, and ignoreText cannot be applied in code. You should embed them directly in the HTML code
