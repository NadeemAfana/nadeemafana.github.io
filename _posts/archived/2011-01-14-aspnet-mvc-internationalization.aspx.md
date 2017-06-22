---
layout: post
title: ASP.NET MVC 5 Internationalization
redirect_from:
- "/post/aspnet-mvc-internationalization.aspx.html"
- "/post/aspnet-mvc-internationalization.aspx/"
tags: [aspnetmvc, aspnet, mvc, i18n, g11n]
---
**[Download Code](/attachments/posts/archived/aspnet-mvc-internationalization.zip)**

### Introduction

If your website targets users from different parts of the world, these users might like to see your website content in their own language. Creating a multilingual website is not an easy task, but it will certainly allow your site to reach more audience. Fortunately, the .NET Framework already has components that support different languages and cultures.

We will build an ASP.NET MVC 5 web application that contains the following features:

*    It can display contents in different languages.
*    It autodetects the language from the user's browser.
*    It allows the user to override the language of their browser.

### Globalization and Localization in ASP.NET

Internationalization involves Globalization and Localization. Globalization is the process of designing applications that support different cultures. Localization is the process of customizing an application for a given culture.

The format for the culture name is `<languagecode2>-<country/regioncode2>`, where `<languagecode2>` is the language code and `<country/regioncode2>` is the subculture code. Examples include es-CL for Spanish (Chile) and en-US for English (United States).

Anyway, Internationalization is often abbreviated to "I18N". The abbreviation takes the first and last letters and the number of letters between them, so 18 stands for the number of letters between the first "I" and the last "N". The same applies to Globalization (G11N), and Localization (L10N).

ASP.NET keeps track of two culture values, the Culture and UICulture. The culture value determines the results of culture-dependent functions, such as the date, number, and currency formatting. The UICulture determines which resources are to be loaded for the page by the ResourceManager. The ResourceManager simply looks up culture-specific resources that is determined by CurrentUICulture. Every thread in .NET has CurrentCulture and CurrentUICulture objects. So ASP.NET inspects these values when rendering culture-dependent functions. For example, if current thread's culture (CurrentCulture) is set to "en-US" (English, United States), DateTime.Now.ToLongDateString() shows "Saturday, January 08, 2011", but if CurrentCulture is set to "es-CL" (Spanish, Chile) the result will be "sábado, 08 de enero de 2011".

Now, let’s review the terms used so far:

*    **Globalization** (G11N): The process of making an application support different languages and regions.
*    **Localization** (L10N): The process of customizing an application for a given language and region.
*    **Internationalization** (I18N): Describes both globalization and localization.
*    **Culture**: It is a language and, optionally, a region.
*    **Locale**: A locale is the same as a culture.
*    **Neutral culture**: A culture that has a specified language, but not a region. (e.g. "en", "es")
*    **Specific culture**: A culture that has a specified language and region. (e.g. "en-US", "en-GB", "es-CL")

### Why do we need a region? Isn’t a language alone enough?

You might not need a region at all. It is true that English in the United States is not the same as English in the United Kingdom but if your application just shows English text readable to people from these English-speaking countries, you will not need a region. The problem arises when you need to deal with numbers, dates, and currencies. For example, compare the following output for two different Spanish-speaking regions (Chile, Mexico):

```csharp
int value = 5600;
 
Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo("es-CL");
Console.WriteLine(DateTime.Now.ToShortDateString());
Console.WriteLine(value.ToString("c"));
 
Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo("es-MX");
Console.WriteLine(DateTime.Now.ToShortDateString());
Console.WriteLine(value.ToString("c"));
 
// Output
26-07-2011 // Date in es-CL, Spanish (Chile)
$5.600,00 // Currency in es-CL, Spanish (Chile)
 
26/07/2011 // Date in es-MX, Spanish (Mexico)
$5,600.00 // Currency in es-MX, Spanish (Mexico)
```

You can notice the difference in date and currency format. The decimal separator in each region is different and can confuse people in the other region. If a Mexican user types one thousand in their culture "1,000", it will be interpreted as 1 (one) in a Chilean culture website. We mainly need regions for this type of reasons and not much for the language itself.

### How to Support Different Languages in ASP.NET MVC

There are two ways to incorporate different languages and cultures in ASP.NET MVC:
1.    By using resource strings in all our site views.
2.    By using different set of views for every language and locale.
3.    By mixing between 1 and 2

### Which one is the best?
It is a matter of convenience. Some people prefer to use a single view for all languages because it is more maintainable. While others think replacing views content with code like "@Resources.Something" might clutter the views and will become unreadable. Some project requirements force developers to implement different views per language. But sometimes you have no choice where layout has to be different like right-to-left languages. Even if you set dir="rtl", this may not be enough in real applications unless the project’s UI layout is really simple. Perhaps, a mix of the two is the best. Anyway, for this example, it makes sense to use resources since we won’t have any issue with the layout for the Spanish, English, and Arabic languages that we will use.

### How can ASP.NET guess the user’s language?

On each HTTP request, there is a header field called Accept-Language which determines which languages the user’s browser supports:
```
Accept-Language: en-us,en;q=0.5
```

This means that my browser prefers English (United States), but it can accept other types of English. The "q" parameter indicates an estimate of the user’s preference for that language. You can control the list of languages using your web browser.

![](/images/posts/archived/aspnet-mvc-internationalization-1.png)
#### IE

![](/images/posts/archived/aspnet-mvc-internationalization-2.png)
#### FireFox

### Globalizing our Web Site

We will create a new ASP.NET MVC web application and globalize it step by step.

Click "File->New Project" menu command within Visual Studio to create a new ASP.NET MVC 5 Project. We'll create a new project using the "MVC" template.

![](/images/posts/archived/aspnet-mvc-internationalization-3.png)
![](/images/posts/archived/aspnet-mvc-internationalization-4.png)


### Creating the Model

We'll need a model to create our web application. Add a class named "Person" to the "Models" folder:

```csharp
public class Person
{
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public int Age { get; set; }
        public string Email { get; set; }
        public string Biography { get; set; }
}
```

### Internationalizing Validation Messages

Our model presented above contains no validation logic, and this is not the case in normal applications nowadays. We can use data annotation attributes to add some validation logic to our model. However, in order to globalize validation messages, we need to specify a few extra parameters. The "ErrorMessageResourceType" indicates the type of resource to look up the error message. "ErrorMessageResourceName" indicates the resource name to lookup the error message. Resource manager will pick the correct resource file based on the current culture.

Now modify the "Person" class and add the following attributes:

```csharp
public class Person
{
    [Display(Name = "FirstName", ResourceType = typeof(Resources.Resources))]    
    [Required(ErrorMessageResourceType = typeof(Resources.Resources),
              ErrorMessageResourceName = "FirstNameRequired")]
    [StringLength(50, ErrorMessageResourceType = typeof(Resources.Resources),
                      ErrorMessageResourceName = "FirstNameLong")]
    public string FirstName { get; set; }
    [Display(Name = "LastName", ResourceType = typeof(Resources.Resources))]    
    [Required(ErrorMessageResourceType = typeof(Resources.Resources),
              ErrorMessageResourceName = "LastNameRequired")]
    [StringLength(50, ErrorMessageResourceType = typeof(Resources.Resources),
                      ErrorMessageResourceName = "LastNameLong")]
    public string LastName { get; set; }
    [Display(Name = "Age", ResourceType = typeof(Resources.Resources))]    
    [Required(ErrorMessageResourceType = typeof(Resources.Resources),
              ErrorMessageResourceName = "AgeRequired")]
    [Range(0, 130, ErrorMessageResourceType = typeof(Resources.Resources),
                   ErrorMessageResourceName = "AgeRange")]
    public int Age { get; set; }
    [Display(Name = "Email", ResourceType = typeof(Resources.Resources))]    
    [Required(ErrorMessageResourceType = typeof(Resources.Resources),
              ErrorMessageResourceName = "EmailRequired")]
    [RegularExpression(".+@.+\\..+", ErrorMessageResourceType = typeof(Resources.Resources),
                                     ErrorMessageResourceName = "EmailInvalid")]
    public string Email { get; set; }
    [Display(Name = "Biography", ResourceType = typeof(Resources.Resources))]    
    public string Biography { get; set; }
}
```

### Localizing Data Annotations Validation Messages

Because we need to perform data validation on our model using Data Annotations, we will have to add translated resource strings for every culture our site will support. In this case, English, Spanish, and Arabic.

We will store resource files in a separate assembly, so we can reference them in other project types in the future.

Right click on the Solution and then choose the "Add->New Project" context menu command. Choose "Class Library" project type and name it "Resources".

Now right click on "Resources" project and then choose "Add->New Item" context menu command. Choose "Resource File" and name it "Resources.resx". This will be our default culture (en-US) since it has no special endings. Add the following names and values to the file like below:


![](/images/posts/archived/aspnet-mvc-internationalization-5.png)

![](/images/posts/archived/aspnet-mvc-internationalization-6.png)

Remember to mark the resource's access modifier property to "public", so it will be accessible from other projects.

Now create a new resource file and name it "Resources.es.resx" and add the following names and values like below:

![](/images/posts/archived/aspnet-mvc-internationalization-7.png)

Now, do the same for the Arabic version. You may not be able to enter the correct strings by keyboard because your OS may not be configured to accept Arabic. However, you can download the files from the link at the top. Anyway, the resource file is included for reference:

![](/images/posts/archived/aspnet-mvc-internationalization-8.png)

We need to reference "Resources" project from our web application, so that we can read the resource strings right from our web site. Right click on "References" under our web project "MvcInternationalization", and choose the "Resources" project from Projects tab.

### Views

We need to extract the English text from all the views and move it to the resource files. Here is a trick, instead of typing the namespace name each time (e.g. "@Resources.Resources.LogOn "), we can add this namespace to the views Web.config and type "@Resources.LogOn" instead. Open the Web.config file under the views folder.

![](/images/posts/archived/aspnet-mvc-internationalization-9.png)

### Determining Culture

There is a header field called "Accept-Language" that the browser sends on every request. This field contains a list of culture names (language-country) that the user has configured in their browser. The problem is that this culture may not reflect the real user's preferred language, such as a computer in a public place. We should allow the user to choose a language explicitly and allow them even to change it. In order to do this sort of things, we need to store the user's preferred language in a store, which can be perfectly a cookie. We will create a base controller that inspects the cookie contents first, if there is no cookie, we will use the "Accept-Language" field sent by their browser. Create a controller and name it "BaseController" like below:

```csharp
public class BaseController : Controller
{
    protected override IAsyncResult BeginExecuteCore(AsyncCallback callback, object state)
    {
        string cultureName = null;
         
        // Attempt to read the culture cookie from Request
        HttpCookie cultureCookie = Request.Cookies["_culture"];
        if (cultureCookie != null)
            cultureName = cultureCookie.Value;
        else
            cultureName = Request.UserLanguages != null && Request.UserLanguages.Length > 0 ? 
                    Request.UserLanguages[0] :  // obtain it from HTTP header AcceptLanguages
                    null;
        // Validate culture name
        cultureName = CultureHelper.GetImplementedCulture(cultureName); // This is safe
         
        // Modify current thread's cultures            
        Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo(cultureName);
        Thread.CurrentThread.CurrentUICulture = Thread.CurrentThread.CurrentCulture;
         
        return base.BeginExecuteCore(callback, state);
    }
}
```

Make sure all controllers in this project inherit from this BaseController. The base controller checks if the cookie exists, and sets the current thread cultures to that cookie value. Of course, because cookie content can be manipulated on the client side, we should always validate its value using a helper class called "CultureHelper". If the culture name is not valid, the helper class returns the default culture.

### CultureHelper Class

The CultureHelper is basically a utility that allows us to store culture names we are implementing in our site:

![](/images/posts/archived/aspnet-mvc-internationalization-10.png)

```csharp
public static class CultureHelper
{
    // Valid cultures
    private static readonly List<string> _validCultures = new List<string> { "af", "af-ZA", "sq", "sq-AL", "gsw-FR", "am-ET", "ar", "ar-DZ", "ar-BH", "ar-EG", "ar-IQ", "ar-JO", "ar-KW", "ar-LB", "ar-LY", "ar-MA", "ar-OM", "ar-QA", "ar-SA", "ar-SY", "ar-TN", "ar-AE", "ar-YE", "hy", "hy-AM", "as-IN", "az", "az-Cyrl-AZ", "az-Latn-AZ", "ba-RU", "eu", "eu-ES", "be", "be-BY", "bn-BD", "bn-IN", "bs-Cyrl-BA", "bs-Latn-BA", "br-FR", "bg", "bg-BG", "ca", "ca-ES", "zh-HK", "zh-MO", "zh-CN", "zh-Hans", "zh-SG", "zh-TW", "zh-Hant", "co-FR", "hr", "hr-HR", "hr-BA", "cs", "cs-CZ", "da", "da-DK", "prs-AF", "div", "div-MV", "nl", "nl-BE", "nl-NL", "en", "en-AU", "en-BZ", "en-CA", "en-029", "en-IN", "en-IE", "en-JM", "en-MY", "en-NZ", "en-PH", "en-SG", "en-ZA", "en-TT", "en-GB", "en-US", "en-ZW", "et", "et-EE", "fo", "fo-FO", "fil-PH", "fi", "fi-FI", "fr", "fr-BE", "fr-CA", "fr-FR", "fr-LU", "fr-MC", "fr-CH", "fy-NL", "gl", "gl-ES", "ka", "ka-GE", "de", "de-AT", "de-DE", "de-LI", "de-LU", "de-CH", "el", "el-GR", "kl-GL", "gu", "gu-IN", "ha-Latn-NG", "he", "he-IL", "hi", "hi-IN", "hu", "hu-HU", "is", "is-IS", "ig-NG", "id", "id-ID", "iu-Latn-CA", "iu-Cans-CA", "ga-IE", "xh-ZA", "zu-ZA", "it", "it-IT", "it-CH", "ja", "ja-JP", "kn", "kn-IN", "kk", "kk-KZ", "km-KH", "qut-GT", "rw-RW", "sw", "sw-KE", "kok", "kok-IN", "ko", "ko-KR", "ky", "ky-KG", "lo-LA", "lv", "lv-LV", "lt", "lt-LT", "wee-DE", "lb-LU", "mk", "mk-MK", "ms", "ms-BN", "ms-MY", "ml-IN", "mt-MT", "mi-NZ", "arn-CL", "mr", "mr-IN", "moh-CA", "mn", "mn-MN", "mn-Mong-CN", "ne-NP", "no", "nb-NO", "nn-NO", "oc-FR", "or-IN", "ps-AF", "fa", "fa-IR", "pl", "pl-PL", "pt", "pt-BR", "pt-PT", "pa", "pa-IN", "quz-BO", "quz-EC", "quz-PE", "ro", "ro-RO", "rm-CH", "ru", "ru-RU", "smn-FI", "smj-NO", "smj-SE", "se-FI", "se-NO", "se-SE", "sms-FI", "sma-NO", "sma-SE", "sa", "sa-IN", "sr", "sr-Cyrl-BA", "sr-Cyrl-SP", "sr-Latn-BA", "sr-Latn-SP", "nso-ZA", "tn-ZA", "si-LK", "sk", "sk-SK", "sl", "sl-SI", "es", "es-AR", "es-BO", "es-CL", "es-CO", "es-CR", "es-DO", "es-EC", "es-SV", "es-GT", "es-HN", "es-MX", "es-NI", "es-PA", "es-PY", "es-PE", "es-PR", "es-ES", "es-US", "es-UY", "es-VE", "sv", "sv-FI", "sv-SE", "syr", "syr-SY", "tg-Cyrl-TJ", "tzm-Latn-DZ", "ta", "ta-IN", "tt", "tt-RU", "te", "te-IN", "th", "th-TH", "bo-CN", "tr", "tr-TR", "tk-TM", "ug-CN", "uk", "uk-UA", "wen-DE", "ur", "ur-PK", "uz", "uz-Cyrl-UZ", "uz-Latn-UZ", "vi", "vi-VN", "cy-GB", "wo-SN", "sah-RU", "ii-CN", "yo-NG" };
    // Include ONLY cultures you are implementing
    private static readonly List<string> _cultures = new List<string> {
        "en-US",  // first culture is the DEFAULT
        "es", // Spanish NEUTRAL culture
        "ar"  // Arabic NEUTRAL culture
        
    };
    /// <summary>
    /// Returns true if the language is a right-to-left language. Otherwise, false.
    /// </summary>
    public static bool IsRighToLeft()
    {
        return System.Threading.Thread.CurrentThread.CurrentCulture.TextInfo.IsRightToLeft;
 
    }
    /// <summary>
    /// Returns a valid culture name based on "name" parameter. If "name" is not valid, it returns the default culture "en-US"
    /// </summary>
    /// <param name="name" />Culture's name (e.g. en-US)</param>
    public static string GetImplementedCulture(string name)
    {
        // make sure it's not null
        if (string.IsNullOrEmpty(name))
            return GetDefaultCulture(); // return Default culture
        // make sure it is a valid culture first
        if (_validCultures.Where(c => c.Equals(name, StringComparison.InvariantCultureIgnoreCase)).Count() == 0)
            return GetDefaultCulture(); // return Default culture if it is invalid
        // if it is implemented, accept it
        if (_cultures.Where(c => c.Equals(name, StringComparison.InvariantCultureIgnoreCase)).Count() > 0)
            return name; // accept it
        // Find a close match. For example, if you have "en-US" defined and the user requests "en-GB", 
        // the function will return closes match that is "en-US" because at least the language is the same (ie English)  
        var n = GetNeutralCulture(name);
        foreach (var c in _cultures)
            if (c.StartsWith(n))
                return c;
        // else 
        // It is not implemented
        return GetDefaultCulture(); // return Default culture as no match found
    }
    /// <summary>
    /// Returns default culture name which is the first name decalared (e.g. en-US)
    /// </summary>
    /// <returns></returns>
    public static string GetDefaultCulture()
    {
        return _cultures[0]; // return Default culture
    }
    public static string GetCurrentCulture()
    {
        return Thread.CurrentThread.CurrentCulture.Name;
    }
    public static string GetCurrentNeutralCulture()
    {
        return GetNeutralCulture(Thread.CurrentThread.CurrentCulture.Name);
    }
    public static string GetNeutralCulture(string name)
    {
        if (!name.Contains("-")) return name;
             
        return name.Split('-')[0]; // Read first part only. E.g. "en", "es"
    }
}
```

We should populate "_cultures" manually. The "_cultures" dictionary stores the list of culture names our site supports. The nice part of this utility class is that it serves similar cultures. For example, if a user is visiting our site from the United Kingdom (en-GB), a culture which is not implemented in our site, he or she will see our site in English using "en-US" (English, United States). This way, you don't have to implement all cultures unless you really care about currency, date format, etc. One important thing to mention is that we added all specific cultures for the Spanish language. For example, we are implementing "es" Spanish in general without any region where a date looks like this "30/07/2011". Now suppose a nerd is coming from Chile (es-CL), it would be very nice display dates in their culture format (30-07-2011) instead of the neutral one. It does not matter if we don’t have a resource file for any of these cultures. ResourceManager can pick a neutral culture when it cannot find a specific culture file, this automatic mechanism is called fallback.

### Controllers

Visual Studio has created a controller named "HomeCotnroller" for us, so we'll use it for simplicity. Modify the "HomeController.cs" so that it looks like below:

```csharp
public class HomeController : BaseController
{
    [HttpGet]
    public ActionResult Index()
    {
        return View();
    }
    [HttpPost]
    public ActionResult Index(Person per)
    {
        return View();
    }
    public ActionResult SetCulture(string culture)
    {
        // Validate input
        culture = CultureHelper.GetImplementedCulture(culture);
        // Save culture in a cookie
        HttpCookie cookie = Request.Cookies["_culture"];
        if (cookie != null)
            cookie.Value = culture;   // update cookie value
        else
        {
            cookie = new HttpCookie("_culture");                
            cookie.Value = culture;
            cookie.Expires = DateTime.Now.AddYears(1);
        }
        Response.Cookies.Add(cookie);
        return RedirectToAction("Index");
    }                
 
}
```

The "SetCulture" action allows the user to change their current culture and stores it in a cookie called "_culture". We are not restricted to cookies, we could instead save the culture name in Session or elsewhere, but cookies are really lightweight since they do not take any type of space on server side.

### Creating a View Template

Now we will implement the View associated with the HomeController's Index action. First delete the existing Index.cshtml file under "Views/Home" folder. Now to implement the view right-click within the "HomeController.Index()" method and select the "Add View" command to create a view template for our home page:


![](/images/posts/archived/aspnet-mvc-internationalization-11.png)

Click OK and replace the existing view if any. When we click the "Add" button, a view template of our "Create" view (which renders the form) is created. Modify it so it looks like below:

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
@model MvcInternationalization.Models.Person&#10;@{&#10;    ViewBag.Title = Resources.AddPerson;&#10;    var culture = System.Threading.Thread.CurrentThread.CurrentUICulture.Name.ToLowerInvariant();&#10;}&#10;@helper selected(string c, string culture)&#10;{&#10;    if (c == culture)&#10;    {&#10;        @:checked=&#34;checked&#34;&#10;    }&#10;}&#10;&lt;h2&gt;@Resources.AddPerson&lt;/h2&gt;&#10;@using(Html.BeginForm(&#34;SetCulture&#34;, &#34;Home&#34;))&#10;{&#10;    &lt;fieldset&gt;&#10;        &lt;legend&gt;@Resources.ChooseYourLanguage&lt;/legend&gt;&#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;en-us&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;en-us&#34; value=&#34;en-us&#34; type=&#34;radio&#34; @selected(&#34;en-us&#34;, culture) /&gt; English&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;es&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;es&#34; value=&#34;es&#34; type=&#34;radio&#34; @selected(&#34;es&#34;, culture) /&gt; Espa&#241;ol&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;control-group&#34;&gt;&#10;            &lt;div class=&#34;controls&#34;&gt;&#10;                &lt;label for=&#34;ar&#34;&gt;&#10;                    &lt;input name=&#34;culture&#34; id=&#34;ar&#34; value=&#34;ar&#34; type=&#34;radio&#34; @selected(&#34;ar&#34;, culture) /&gt; &#1575;&#1604;&#1593;&#1585;&#1576;&#1610;&#1577;&#10;                &lt;/label&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;       &#10;    &lt;/fieldset&gt;&#10;        &#10;     &#10;     &#10;}&#10;@using (Html.BeginForm()) &#10;{&#10;    @Html.AntiForgeryToken()&#10;     &#10;    &lt;div class=&#34;form-horizontal&#34;&gt;        &#10;        &lt;hr /&gt;&#10;        @Html.ValidationSummary(true)&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            @Html.LabelFor(model =&gt; model.FirstName, new { @class = &#34;control-label col-md-2&#34; })&#10;            &lt;div class=&#34;col-md-10&#34;&gt;&#10;                @Html.EditorFor(model =&gt; model.FirstName)&#10;                @Html.ValidationMessageFor(model =&gt; model.FirstName)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            @Html.LabelFor(model =&gt; model.LastName, new { @class = &#34;control-label col-md-2&#34; })&#10;            &lt;div class=&#34;col-md-10&#34;&gt;&#10;                @Html.EditorFor(model =&gt; model.LastName)&#10;                @Html.ValidationMessageFor(model =&gt; model.LastName)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            @Html.LabelFor(model =&gt; model.Age, new { @class = &#34;control-label col-md-2&#34; })&#10;            &lt;div class=&#34;col-md-10&#34;&gt;&#10;                @Html.EditorFor(model =&gt; model.Age)&#10;                @Html.ValidationMessageFor(model =&gt; model.Age)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            @Html.LabelFor(model =&gt; model.Email, new { @class = &#34;control-label col-md-2&#34; })&#10;            &lt;div class=&#34;col-md-10&#34;&gt;&#10;                @Html.EditorFor(model =&gt; model.Email)&#10;                @Html.ValidationMessageFor(model =&gt; model.Email)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            @Html.LabelFor(model =&gt; model.Biography, new { @class = &#34;control-label col-md-2&#34; })&#10;            &lt;div class=&#34;col-md-10&#34;&gt;&#10;                @Html.EditorFor(model =&gt; model.Biography)&#10;                @Html.ValidationMessageFor(model =&gt; model.Biography)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;        &lt;div class=&#34;form-group&#34;&gt;&#10;            &lt;div class=&#34;col-md-offset-2 col-md-10&#34;&gt;&#10;                &lt;input type=&#34;submit&#34; value=&#34;@Resources.Create&#34; class=&#34;btn btn-default&#34; /&gt;&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;    &lt;/div&gt;&#10;}&#10;@section Scripts {&#10;    @Scripts.Render(&#34;~/bundles/jqueryval&#34;)&#10;    &lt;script type=&#34;text/javascript&#34;&gt;&#10;        (function ($) {&#10;            $(&#34;input[type = 'radio']&#34;).click(function () {&#10;                $(this).parents(&#34;form&#34;).submit(); // post form&#10;            });&#10;             &#10;        })(jQuery);&#10;    &lt;/script&gt;&#10;}
</pre>

The javascript code simply post back the form to set the culture. The "selected" helper is used to mark the appropriate culture radio button as checked.

Of course, we should not forget about partial views too

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
@using Microsoft.AspNet.Identity&#10;@if (Request.IsAuthenticated)&#10;{&#10;    using (Html.BeginForm(&#34;LogOff&#34;, &#34;Account&#34;, FormMethod.Post, new { id = &#34;logoutForm&#34;, @class = &#34;navbar-right&#34; }))&#10;    {&#10;    @Html.AntiForgeryToken()&#10;    &lt;ul class=&#34;nav navbar-nav navbar-right&#34;&gt;&#10;        &lt;li&gt;&#10;            @Html.ActionLink(User.Identity.GetUserName(), &#34;Manage&#34;, &#34;Account&#34;, routeValues: null, htmlAttributes: new { title = &#34;Manage&#34; })&#10;        &lt;/li&gt;&#10;        &lt;li&gt;&lt;a href=&#34;javascript:document.getElementById('logoutForm').submit()&#34;&gt;@Resources.LogOff&lt;/a&gt;&lt;/li&gt;&#10;    &lt;/ul&gt;&#10;    }&#10;}&#10;else&#10;{&#10;    &lt;ul class=&#34;nav navbar-nav navbar-right&#34;&gt;&#10;        &lt;li&gt;@Html.ActionLink(Resources.Register, &#34;Register&#34;, &#34;Account&#34;, routeValues: null, htmlAttributes: new { id = &#34;registerLink&#34; })&lt;/li&gt;&#10;        &lt;li&gt;@Html.ActionLink(Resources.LogOn, &#34;Login&#34;, &#34;Account&#34;, routeValues: null, htmlAttributes: new { id = &#34;loginLink&#34; })&lt;/li&gt;&#10;    &lt;/ul&gt;&#10;}
</pre>

### Left-to-right or Right-to-left

HTML supports rtl languages too, so we need to make sure our main HTML tag has the appropriate direction. Modify the _Layout.cshtml file to look like the following:

<pre markdown="0" class="brush: csharp; class-name: 'razor'; toolbar: false;">
&lt;!DOCTYPE html&gt;
&#10;&lt;html lang=&#34;@CultureHelper.GetCurrentNeutralCulture()&#34; dir=&#34;@(CultureHelper.IsRighToLeft() ? &#34;rtl&#34; : &#34;ltr&#34;)&#34;&gt;&#10;&lt;head&gt;&#10;    &lt;meta charset=&#34;utf-8&#34; /&gt;&#10;    &lt;meta name=&#34;viewport&#34; content=&#34;width=device-width, initial-scale=1.0&#34;&gt;&#10;    &lt;title&gt;@ViewBag.Title - ASP.NET MVC Internationalization&lt;/title&gt;&#10;    @Styles.Render(&#34;~/Content/css&#34; + (CultureHelper.IsRighToLeft() ? &#34;-rtl&#34; : &#34;&#34;))&#10;    @Scripts.Render(&#34;~/bundles/modernizr&#34;)&#10;&lt;/head&gt;&#10;&lt;body&gt;&#10;    &lt;div class=&#34;navbar navbar-inverse navbar-fixed-top&#34;&gt;&#10;        &lt;div class=&#34;container&#34;&gt;&#10;            &lt;div class=&#34;navbar-header&#34;&gt;&#10;                &lt;button type=&#34;button&#34; class=&#34;navbar-toggle&#34; data-toggle=&#34;collapse&#34; data-target=&#34;.navbar-collapse&#34;&gt;&#10;                    &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;                    &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;                    &lt;span class=&#34;icon-bar&#34;&gt;&lt;/span&gt;&#10;                &lt;/button&gt;&#10;                @Html.ActionLink(&#34;ASP.NET MVC Internationalization&#34;, &#34;Index&#34;, &#34;Home&#34;, null, new { @class = &#34;navbar-brand&#34; })&#10;            &lt;/div&gt;&#10;            &lt;div class=&#34;navbar-collapse collapse&#34;&gt;&#10;                &lt;ul class=&#34;nav navbar-nav&#34;&gt;                    &#10;                &lt;/ul&gt;&#10;                @Html.Partial(&#34;_LoginPartial&#34;)&#10;            &lt;/div&gt;&#10;        &lt;/div&gt;&#10;    &lt;/div&gt;&#10;    &lt;div class=&#34;container body-content&#34;&gt;&#10;        @RenderBody()&#10;        &lt;hr /&gt;&#10;        &lt;footer&gt;&#10;            &lt;p&gt;@DateTime.Now&lt;/p&gt;&#10;        &lt;/footer&gt;&#10;    &lt;/div&gt;&#10;    @Scripts.Render(&#34;~/bundles/jquery&#34;)&#10;    @Scripts.Render(&#34;~/bundles/bootstrap&#34; + (CultureHelper.IsRighToLeft() ? &#34;-rtl&#34; : &#34;&#34;))&#10;    @RenderSection(&#34;scripts&#34;, required: false)&#10;&lt;/body&gt;&#10;&lt;/html&gt;
</pre>

Now we need to provide basically two sets of CSS and JS files: One for left-to-right languages and one for right-to-left languages. Because MVC default template is using Bootstrap, we need to install the RTL version of bootstrap files. For this, open the package manager console (aka Nuget) by choosing Tools -> Library Package Manager -> Package Manager Console and type:

```
Install-Package Twitter.Bootstrap.RTL
```

Make sure you have the two files bootstrap-rtl.js and bootstrap-rtl.css

We need to create two bundles: one for RTL and one for LTR. Modify the file BundleConfig.cs and add the following:


```csharp
bundles.Add(new ScriptBundle("~/bundles/bootstrap-rtl").Include(
           "~/Scripts/bootstrap-rtl.js",
           "~/Scripts/respond.js"));
 
 bundles.Add(new StyleBundle("~/Content/css-rtl").Include(
           "~/Content/bootstrap-rtl.css",
           "~/Content/site.css"));
```

### Try It Out

Run the website now. Notice that client side validation is working nicely. Click on radio buttons to switch between cultures, and notice how right-to-left language is showing correctly. Using separate views allowed us to control how to position elements, and have made our views clean and readable.

![](/images/posts/archived/aspnet-mvc-internationalization-12.png)
#### English
![](/images/posts/archived/aspnet-mvc-internationalization-13.png)
#### Spanish
![](/images/posts/archived/aspnet-mvc-internationalization-14.png)
#### Arabic


### How to store culture in the URL instead of a cookie?

There can be different reasons why you want the culture to be part of your website url (such as search engine indexing). Anyway. First let's fist define the culture to be part of our routes. Edit RouteConfig.cs like below:

```csharp
public static void RegisterRoutes(RouteCollection routes)
{
    routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
    routes.MapRoute(
        name: "Default",
        url: "{culture}/{controller}/{action}/{id}",
        defaults: new {culture = CultureHelper.GetDefaultCulture(), controller = "Home", action = "Index", id = UrlParameter.Optional }
    );
}
```

Notice the we used the default culture in case it is missing. Now modify the base controller:

```csharp
protected override IAsyncResult BeginExecuteCore(AsyncCallback callback, object state)
{
    string cultureName = RouteData.Values["culture"] as string; 
 
    // Attempt to read the culture cookie from Request
    if (cultureName == null)               
        cultureName = Request.UserLanguages != null && Request.UserLanguages.Length > 0 ? Request.UserLanguages[0] : null; // obtain it from HTTP header AcceptLanguages
 
    // Validate culture name
    cultureName = CultureHelper.GetImplementedCulture(cultureName); // This is safe
 
 
    if (RouteData.Values["culture"] as string != cultureName) {
         
        // Force a valid culture in the URL
        RouteData.Values["culture"] = cultureName.ToLowerInvariant(); // lower case too
 
        // Redirect user
        Response.RedirectToRoute(RouteData.Values);                
    }
   
 
    // Modify current thread's cultures            
    Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo(cultureName);
    Thread.CurrentThread.CurrentUICulture = Thread.CurrentThread.CurrentCulture;
 
 
    return base.BeginExecuteCore(callback, state);
}
```

The final step is to modify the `SetCulture` action in `HomeController.cs`

```csharp
public ActionResult SetCulture(string culture)
{
    // Validate input
    culture = CultureHelper.GetImplementedCulture(culture);
    RouteData.Values["culture"] = culture;  // set culture
 
    
    return RedirectToAction("Index");
}     
```
**NOTE**: To force the default culture appear in the URL, simply set the default value for culture in RouteConfig.cs to string.Empty

![](/images/posts/archived/aspnet-mvc-internationalization-15.png)

### Summary

Building a multilingual web application is not an easy task. but it's worth it especially for web applications targeting users from all over the world, something which many sites do. It is true that globalization is not the first priority in site development process, however, it should be well planned early in the stage of development so it can be easily implemented in the future. Luckily, ASP.NET supports globalization and there are plenty of .NET classes that are handy. We have seen how to create an ASP.NET MVC application that supports 3 different languages, including a right-to-left one, which requires a different UI layout. Anyway, here is a summary of how to globalize a site in ASP.NET MVC:

 
1.    Add a base controller from which all controllers inherit. This controller will intercept the view names returned and will adjust them depending on the current culture set.
2.    Add a helper class that stores the list of culture names that the site will support.
3.    Create resource files that contain translation of all string messages. (e.g. Resources.resx, Resources.es.resx, Resources.ar.resx, etc )
4.    Update views to use localized text.
5.    Localize javascript files.


I hope this helps.

Any questions or comments are welcome.