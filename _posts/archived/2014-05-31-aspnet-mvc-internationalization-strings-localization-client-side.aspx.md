---
layout: post
title: ASP.NET MVC 5 Internationalization · Strings Localization on the client-side
redirect_from:
- "/post/aspnet-mvc-internationalization-strings-localization-client-side.aspx.html"
tags: [asp, .net, localization, i18n, g10n]
---
In this post, I will show a technique on how to localize and use strings on the client-side based on the previous article [ASP.NET MVC Internationalization](/post/aspnet-mvc-internationalization.aspx).

Nowadays, most web applications rely heavily on Javascript, and it is very likely that a web application needs to display some messages to the end user using Javascript whether for validation or notification purpose.

One way of solving this problem is by creating a unique javascript file that contains all the localization strings per culture (e.g. resouces.en-us.js, resources.es.js), similar to the .resx files. This method violates the DRY principle and is hard to maintain. Adding one extra resource requires all the javascript files to be updated. 

A better solution is to get the strings from the server-side dynamically. This conforms with the DRY principle. All the resources come from one single place (resx, database, xml, etc). See [How to Store Strings in a Database or Xml](/post/aspnet-mvc-internationalization-store-strings-in-database-or-xml.aspx)

## How to get the strings dynamically?
By adding a new controller and action, it is possible to return all the needed resources using ajax.

Add a new `Controller` called `ResourceController` and a new action called `GetResources`

```csharp
using Resource = Resources.Resources;
namespace MvcInternationalization.Controllers
{
    public class ResourceController : BaseController
    {
         
        public JsonResult GetResources()
        {
           return Json(
                typeof(Resource)
                .GetProperties()
                .Where(p => !p.Name.IsLikeAny("ResourceManager", "Culture")) // Skip the properties you don't need on the client side.
                .ToDictionary(p => p.Name, p => p.GetValue(null) as string)
                 , JsonRequestBehavior.AllowGet);
        }
    }
}
```
Or the following equivalent non-reflection version
```csharp
using Resource = Resources.Resources;
namespace MvcInternationalization.Controllers
{
    public class ResourceController : BaseController
    {
         
        public JsonResult GetResources()
        {
            return Json(new Dictionary<string, string> { 
                {"Age", Resource.Age},
                {"FirstName", Resource.FirstName},
                {"LastName", Resource.LastName},
                {"EnterNumber", Resource.EnterNumber}
                    
            }, JsonRequestBehavior.AllowGet);
        }
    }
}
```

Before accessing the localized strings, they need to be fetched from the server and stored in a global variable. I will put the code in `_Layout.cshtml` for illustration purpose. 

```javascript
<script type="text/javascript">
    var resources = {}; // Global variable.
     
    (function ($) {
        $.getJSON("@Url.Action("GetResources", "Resource")", function(data){
            resources = data;
        });
    })(jQuery);
    </script>
```
All the resources now can be accessed via the resources variable in Javascript. Make sure that the data is fetched from the server first before the resources variable is accessed. This can lead to subtle bugs. If the resources are needed when the page loads immediately, then loading all of them from the server in a mater view (eg `_Layout.cshtml`) is recommended. 

If I type `resources` in my browser's console window:
![Resources](/images/posts/archived/aspnet-mvc-internationalization-strings-localization-client-side-1.png "Resources")

## How to localize the annoying message “The field XXX must be a number.”?

![The field XXX must be a number](/images/posts/archived/aspnet-mvc-internationalization-strings-localization-client-side-2.png "The field XXX must be a number")

There is no easy way to localize this text on the server-side besides changing the value of the attribute `data-val-number` manually. However, it is much easier to fix this on the client side.

The following code will not work as you might expect: 
```javascript
var resources = {}; // Global variable.
    (function ($) {
        $.getJSON("@Url.Action("GetResources", "Resource")", function(data){
            resources = data;
            ReplaceInvalidNumberMessage(resources.EnterNumber);
        });
        ReplaceInvalidNumberMessage = function (message) {
            $("form").each(function () {
                var $form = $(this);
                $.each($form.validate().settings.messages, function () {
                    if (this["number"] !== undefined) {
                        this.number = message;
                    }
                });
            });
        }
    })(jQuery);
```
![The field XXX must be a number fixed](/images/posts/archived/aspnet-mvc-internationalization-strings-localization-client-side-3.png "The field XXX must be a number fixed")

![The field XXX must be a number fixed Arabic](/images/posts/archived/aspnet-mvc-internationalization-strings-localization-client-side-4.png "The field XXX must be a number fixed Arabic")

## How to improve performance?

Reading the resources on every page load is acceptable for small scale web applications. However, it can be quite expensive for highly accessed web sites even though the strings are individually cached on the server side.

The solution is output caching. Output caching prevents JSON serialization on each method call. 

The regular output caching does not work because the values vary per culture. The web site ends up serving the same culture for all users. Custom output caching is the way to go. 

```csharp
// GET: /Resource/GetResources
const int durationInSeconds = 2 * 60 * 60;  // 2 hours.
[OutputCache(VaryByCustom = "culture", Duration = durationInSeconds)] 
public JsonResult GetResources()
{
    return Json(
         typeof(Resource)
            .GetProperties()
            .Where(p => !p.Name.IsLikeAny("ResourceManager", "Culture")) // Skip the properties you don't need on the client side.
            .ToDictionary(p => p.Name, p => p.GetValue(null) as string)
         , JsonRequestBehavior.AllowGet);
}
```
I put the duration value in a separate variable on purpose. It’s not obvious whether `Duration` is in seconds or milliseconds unless I hover the mouse over it. 

Custom output caching needs some explicit handling in the `Global.asax.cs` file: 

```csharp
public override string GetVaryByCustomString(HttpContext context, string custom)
{            
    if (custom == "culture") // culture name (e.g. "en-US") is what should vary caching
    {
        string cultureName = null;
               
        // Attempt to read the culture cookie from Request
        HttpCookie cultureCookie = Request.Cookies["_culture"];
        if (cultureCookie != null) {
            cultureName = cultureCookie.Value;
        }
        else {
            cultureName = Request.UserLanguages != null
            && Request.UserLanguages.Length > 0 ? 
            Request.UserLanguages[0] : null; // obtain it from HTTP header AcceptLanguages
        }
                 
        // Validate culture name
        cultureName = CultureHelper.GetImplementedCulture(cultureName);
        return cultureName.ToLower(); // use culture name as the cache key, "es", "en-us", "es-cl", etc.
    }
    return base.GetVaryByCustomString(context, custom);
}
```







