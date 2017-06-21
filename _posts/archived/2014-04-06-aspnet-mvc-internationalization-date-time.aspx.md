---
layout: post
title: ASP.NET MVC 5 Internationalization · Date and Time
redirect_from:
- "/post/aspnet-mvc-internationalization-date-time.aspx/"
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

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
&lt;!DOCTYPE HTML&gt;&#10;&lt;html lang=&#34;@CultureHelper.GetCurrentNeutralCulture()&#34; dir=&#34;@(CultureHelper.IsRightToLeft() ? &#34; rtl&#34;=rtl&#34; = =&#34;ltr&#34; )&#34;=)&#34;&gt;&#10;&lt;head&gt;&#10;        &lt;meta charset=&#34;utf-8&#34; /&gt;&#10;        &lt;meta name=&#34;viewport&#34; content=&#34;width=device-width, initial-scale=1.0&#34; /&gt;&#10;        &lt;title&gt;@ViewBag.Title - ASP.NET MVC Internationalization&lt;/title&gt;&#10;    @Styles.Render(&#34;~/Content/css&#34; + (CultureHelper.IsRightToLeft() ? &#34;-rtl&#34; : &#34;&#34;))&#10;    @Scripts.Render(&#34;~/bundles/modernizr&#34;)&#10;&lt;/head&gt;&#10;&lt;body&gt;&#10;        &lt;div class=&#34;navbar navbar-inverse navbar-fixed-top&#34;&gt;&#10;        &lt;div class=&#34;container&#34;&gt;&#10;        &lt;div class=&#34;navbar-header&#34;&gt;&#10;        &lt;button type=&#34;button&#34; class=&#34;navbar-toggle&#34; data-toggle=&#34;collapse&#34; data-target=&#34;.navbar-collapse&#34;&gt;&#10;        &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;        &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;        &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;                &lt;/button&gt;&#10;                @Html.ActionLink(&#34;ASP.NET MVC Internationalization&#34;, &#34;Index&#34;, &#34;Home&#34;, null, new { @class = &#34;navbar-brand&#34; })&#10;            &lt;/div&gt;&#10;        &lt;div class=&#34;navbar-collapse collapse&#34;&gt;&#10;        &lt;ul class=&#34;nav navbar-nav&#34;&gt;                    &#10;                &lt;/ul&gt;&#10;                @Html.Partial(&#34;_LoginPartial&#34;)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;    &lt;/div&gt;&#10;        &lt;div class=&#34;container body-content&#34;&gt;&#10;        @RenderBody()&#10;        &lt;hr /&gt;&#10;        &lt;footer&gt;           &#10;        &lt;/footer&gt;&#10;    &lt;/div&gt;&#10;    @Scripts.Render(&#34;~/bundles/jquery&#34;)&#10;    @Scripts.Render(&#34;~/bundles/bootstrap&#34; + (CultureHelper.IsRightToLeft() ? &#34;-rtl&#34; : &#34;&#34;))&#10;    @RenderSection(&#34;scripts&#34;, required: false)&#10;        &lt;script type=&#34;text/javascript&#34;&gt;&#10;            function cookieExists(name) {&#10;                var nameToFind = name + &#34;=&#34;;&#10;                var cookies = document.cookie.split(';');&#10;                for (var i = 0; i &lt; cookies.length; i++) {&#10;                    if (cookies[i].trim().indexOf(nameToFind) === 0) return true;&#10;                }&#10;                return false;&#10;            }&#10;            if (!cookieExists(&#34;_timeZoneOffset&#34;)) {&#10;                var now = new Date();&#10;                var timeZoneOffset = -now.getTimezoneOffset();  // in minutes&#10;                now.setTime(now.getTime() + 10*24*60*60*1000); // keep it for 10 days&#10;                document.cookie = &#34;_timeZoneOffset=&#34; + timeZoneOffset.toString() + &#34;;expires=&#34; + now.toGMTString() + &#34;;path=/;&#34; + document.cookie;&#10;                @* Uncomment the following line to force page refresh.  *@&#10;                // window.location.reload();&#10;                }&#10;    &lt;/script&gt;&#10;&lt;/body&gt;&#10;&lt;/html&gt;
</pre>
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

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
@model IEnumerable&lt;DateTimeOffset&gt;&#10;@using MvcInternationalization.Extensions &#10; &#10;@{&#10;    ViewBag.Title = &#34;Date and Time&#34;;&#10;    var culture = System.Threading.Thread.CurrentThread.CurrentUICulture.Name.ToLowerInvariant();&#10;}&#10; &#10;@helper selected(string c, string culture)&#10;{&#10;    if (c == culture)&#10;    {&#10;        @:checked=&#34;checked&#34;&#10;    }&#10;}&#10; &#10;  &#10; &#10;@using(Html.BeginForm(&#34;SetCulture&#34;, &#34;Home&#34;))&#10;{&#10;    &lt;fieldset&gt;&#10;        &lt;legend&gt;@Resources.ChooseYourLanguage&lt;/legend&gt;&#10; &#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;en-us&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;en-us&#34; value=&#34;en-us&#34; type=&#34;radio&#34; @selected(&#34;en-us&#34;, culture) /&gt; English&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10; &#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;es&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;es&#34; value=&#34;es&#34; type=&#34;radio&#34; @selected(&#34;es&#34;, culture) /&gt; Espa&#241;ol&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10; &#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;ar&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;ar&#34; value=&#34;ar&#34; type=&#34;radio&#34; @selected(&#34;ar&#34;, culture) /&gt; &#1575;&#1604;&#1593;&#1585;&#1576;&#1610;&#1577;&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10; &#10;       &#10;    &lt;/fieldset&gt;&#10;}&#10;     &lt;div&gt;&#10;         &lt;table class=&#34;table table-bordered table-hover table-striped&#34;&gt;&#10;             &lt;thead&gt;&#10;                 &lt;tr&gt;&#10;                     &lt;th&gt;UTC&lt;/th&gt;&#10;                     &lt;th&gt;Server&lt;/th&gt;&#10;                     &lt;th&gt;Local &lt;span style=&#34;color: lightgray;&#34;&gt; - Your browser: @ViewBag.TimeZoneOffset&lt;/span&gt;&lt;/th&gt;&#10;                 &lt;/tr&gt;&#10;             &lt;/thead&gt;&#10;             &lt;tbody&gt;&#10;                 @foreach (DateTimeOffset dto in Model) {&#10;                     &lt;tr&gt;&#10;                         &lt;td&gt;@dto.ToUniversalTime().ToString(&#34;g&#34;)&lt;/td&gt;&#10;                         &lt;td&gt;@dto.ToLocalTime().ToString(&#34;g&#34;)&lt;/td&gt; @* You might be surprised that this value is actually local to the machine currently running. *@&#10;                         &lt;td&gt;@dto.ToOffset(ViewBag.TimeZoneOffset).ToString(&#34;g&#34;)  &lt;/td&gt;&#10;                     &lt;/tr&gt;&#10;                 }&#10;             &lt;/tbody&gt;&#10;         &lt;/table&gt;&#10;     &lt;/div&gt;  &#10;     &#10;     &#10; &#10; &#10;  &#10; &#10; &#10;@section Scripts {&#10;    @Scripts.Render(&#34;~/bundles/jqueryval&#34;)&#10; &#10;    &lt;script type=&#34;text/javascript&#34;&gt;&#10;        (function ($) {&#10;            $(&#34;input[type = 'radio']&#34;).click(function () {&#10;                $(this).parents(&#34;form&#34;).submit(); // post form&#10;            });            &#10;             &#10;        })(jQuery);&#10;    &lt;/script&gt;&#10;}
</pre>

### Try it out

![Try it out](/images/posts/archived/aspnet-mvc-internationalization-date-time-1.png "Try it out")