---
layout: post
title: ASP.NET MVC 5 Internationalization · Date and Time
redirect_from:
- "/post/aspnet-mvc-internationalization-date-time.aspx.html"
- "/post/aspnet-mvc-internationalization-date-time.aspx"
tags: [asp, .net, localization, i18n, g10n]
---
**[Download Code](/attachments/posts/archived/aspnet-mvc-internationalization-date-time.zip)**

It would be nice if a web site can show times local to the region where the visitor is located instead of server location (web server, database, etc). For example, a user from Spain that is looking at their order summary on some web site expects that their order date and time `28/01/2014 14:32:26` to be a local (Spain) time, and not a web server time or database server time. Although it is not wrong to add a suffix denoting the region such as `28/01/2014 14:32:26 PST`, it is much more readable for users to see times local to their region.

The .NET Framework has different types and methods that help deal with different time zones which can be quite complicated. The good news is that in general, time zones are not needed in order to display times based on the user’s location. It is simpler than you might think.

There are different ways of dealing with this problem. An easy and effective way is to store all times in the database as UTC (Coordinated Universal Time) and then adjust time (before display) based on where the user is located in the world. The most common .NET type is `DateTime`. A `DateTime` stores a date and optionally a time. It has a `Kind` property which determines if the value is one of three: `local`, `UTC`, or `Unspecified`. 

Another type is called `DateTimeOffset`. A `DateTimeOffset` stores a date and time relative to UTC. Additionally, it stores an offset from UTC as `TimeSpan`. 

```csharp
Console.WriteLine (DateTime.Now);         // 1/28/2014 2:53:50 PM
Console.WriteLine (DateTimeOffset.Now);   // 1/28/2014 2:53:50 PM -08:00
```

`DateTime` and `DateTimeOffset` differ in how they deal with time zones and in comparisons. For i18n, I prefer `DateTimeOffset` because it refers to a more precise point in time. Also, SQL Server 2008 and above support `DateTimeOffset` as a native data type.

## How to capture the user’s time zone offset?
With the aid of the browser, the date time offset can be captured easily using javascript.

```javascript
var timeZoneOffset = -new Date().getTimezoneOffset(); // In minutes.
```

The time zone offset value is also needed on the server side in order to adjust time values before displaying them to the end user. A good place to store this offset is in a cookie. 

```javascript
function cookieExists(name) {
    var nameToFind = name + "=";
    var cookies = document.cookie.split(';');
    for (var i = 0; i < cookies.length; i++) {
        if (cookies[i].trim().indexOf(nameToFind) === 0) return true;
    }
    return false;
}
if (!cookieExists("_timeZoneOffset")) {
    var now = new Date();
    var timeZoneOffset = -now.getTimezoneOffset();  // in minutes
    now.setTime(now.getTime() + 10*24*60*60*1000); // keep it for 10 days
    document.cookie = "_timeZoneOffset=" + timeZoneOffset.toString() 
                  + ";expires=" + now.toGMTString() + ";path=/;" + document.cookie;
    
    // Uncomment the following line to force page refresh.  
    // window.location.reload();
    }
```

The previous javascript code should always be always executed, so it would be a good idea to put this code in a common place such as `_Layout.cshtml` 

```html
<!DOCTYPE HTML>
<html lang="@CultureHelper.GetCurrentNeutralCulture()" dir="@(CultureHelper.IsRightToLeft() ? " rtl"=rtl" = ="ltr" )"=)">
<head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>@ViewBag.Title - ASP.NET MVC Internationalization</title>
    @Styles.Render("~/Content/css" + (CultureHelper.IsRightToLeft() ? "-rtl" : ""))
    @Scripts.Render("~/bundles/modernizr")
</head>
<body>
        <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
        <div class="navbar-header">
        <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
                </button>
                @Html.ActionLink("ASP.NET MVC Internationalization", "Index", "Home", null, new { @class = "navbar-brand" })
            </div>
        <div class="navbar-collapse collapse">
        <ul class="nav navbar-nav">                    
                </ul>
                @Html.Partial("_LoginPartial")
            </div>
        </div>
    </div>
        <div class="container body-content">
        @RenderBody()
        <hr />
        <footer>           
        </footer>
    </div>
    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/bootstrap" + (CultureHelper.IsRightToLeft() ? "-rtl" : ""))
    @RenderSection("scripts", required: false)
        <script type="text/javascript">
            function cookieExists(name) {
                var nameToFind = name + "=";
                var cookies = document.cookie.split(';');
                for (var i = 0; i < cookies.length; i++) {
                    if (cookies[i].trim().indexOf(nameToFind) === 0) return true;
                }
                return false;
            }
            if (!cookieExists("_timeZoneOffset")) {
                var now = new Date();
                var timeZoneOffset = -now.getTimezoneOffset();  // in minutes
                now.setTime(now.getTime() + 10*24*60*60*1000); // keep it for 10 days
                document.cookie = "_timeZoneOffset=" + timeZoneOffset.toString() + ";expires=" + now.toGMTString() + ";path=/;" + document.cookie;
                @* Uncomment the following line to force page refresh.  *@
                // window.location.reload();
                }
    </script>
</body>
</html>
```
In order to convert a `DateTime` to the user local time, an extension method `ToOffset` is needed. `ToOffset` should accept the time zone offset value. 

```csharp
/// <summary>
/// Converts the value of the current DateTime object to the date and time specified by an offset value.
/// </summary>
/// <param name="dt" />DateTime value.</param>
/// <param name="offset" />The offset to convert the DateTime value to.</param>
/// <returns>DateTime value that is local to an offset.</returns>
public static DateTime ToOffset (this DateTime dt, TimeSpan offset)
{
    return dt.ToUniversalTime().Add(offset);
}
```
`DateTimeOffset` already provides a similar method, so there is no need to implement it.

Parse the time zone offset value from the cookie in the base controller, and store it somewhere so it can be used later on the server side.

```csharp
// Parse TimeZoneOffset.
ViewBag.TimeZoneOffset = TimeSpan.FromMinutes(0); // Default offset (Utc) if cookie is missing.
var timeZoneCookie = Request.Cookies["_timeZoneOffset"];
if (timeZoneCookie != null) {
             
    double offsetMinutes = 0;
    if (double.TryParse(timeZoneCookie.Value, out offsetMinutes)) {
        // Store in ViewBag. You can use Session, TempData, or anything else.
        ViewBag.TimeZoneOffset = TimeSpan.FromMinutes(offsetMinutes); 
    }
}
```

The following is a model that illustrates how the date and time display is affected by different time zones. 

```csharp
[HttpGet]
public ActionResult Index()
{
    // Sample data to show dates and times in different places in the world.
    var model = new Collection<DateTimeOffset> { 
        DateTimeOffset.Parse("2014-04-04T20:30:00-7:00"),
        DateTimeOffset.Parse("2014-04-01T11:00:00-2:00"),
        DateTimeOffset.Parse("2014-04-02T00:00:00+2:00"),
    };
    return View(model);
}
```
I used `DateTimeOffset` for illustrative purpose only, `DateTime` would work just fine. 

```html
@model IEnumerable<DateTimeOffset>
@using MvcInternationalization.Extensions 
 
@{
    ViewBag.Title = "Date and Time";
    var culture = System.Threading.Thread.CurrentThread.CurrentUICulture.Name.ToLowerInvariant();
}
 
@helper selected(string c, string culture)
{
    if (c == culture)
    {
        @:checked="checked"
    }
}
 
  
 
@using(Html.BeginForm("SetCulture", "Home"))
{
    <fieldset>
        <legend>@Resources.ChooseYourLanguage</legend>
 
        <div class="control-group">
            <div class="controls">
                <label for="en-us">
                    <input name="culture" id="en-us" value="en-us" type="radio" @selected("en-us", culture) /> English
                </label>
            </div>
        </div>
 
        <div class="control-group">
            <div class="controls">
                <label for="es">
                    <input name="culture" id="es" value="es" type="radio" @selected("es", culture) /> Español
                </label>
            </div>
        </div>
 
        <div class="control-group">
            <div class="controls">
                <label for="ar">
                    <input name="culture" id="ar" value="ar" type="radio" @selected("ar", culture) /> العربية
                </label>
            </div>
        </div>
 
       
    </fieldset>
}
     <div>
         <table class="table table-bordered table-hover table-striped">
             <thead>
                 <tr>
                     <th>UTC</th>
                     <th>Server</th>
                     <th>Local <span style="color: lightgray;"> - Your browser: @ViewBag.TimeZoneOffset</span></th>
                 </tr>
             </thead>
             <tbody>
                 @foreach (DateTimeOffset dto in Model) {
                     <tr>
                         <td>@dto.ToUniversalTime().ToString("g")</td>
                         <td>@dto.ToLocalTime().ToString("g")</td> @* You might be surprised that this value is actually local to the machine currently running. *@
                         <td>@dto.ToOffset(ViewBag.TimeZoneOffset).ToString("g")  </td>
                     </tr>
                 }
             </tbody>
         </table>
     </div>  
     
     
 
 
  
 
 
@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
 
    <script type="text/javascript">
        (function ($) {
            $("input[type = 'radio']").click(function () {
                $(this).parents("form").submit(); // post form
            });            
             
        })(jQuery);
    </script>
}
```

### Try it out

![Try it out](/images/posts/archived/aspnet-mvc-internationalization-date-time-1.png "Try it out")



