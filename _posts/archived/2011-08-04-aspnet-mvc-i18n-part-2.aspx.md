---
layout: post
title: ASP.NET MVC Internationalization - Part 2 (NerdDinner)
redirect_from:
- "/post/aspnet-mvc-internationalization-part-2.aspx.html"
- "/post/aspnet-mvc-internationalization-part-2.aspx/"
tags: [aspnet, i18n, g11n, nerddinner]
---
**[Download Code](https://github.com/NadeemAfana/NerdDinner-Internationalization)**

In my previous blog post, [ASP.NET MVC Internationalization](/post/aspnet-mvc-internationalization.aspx), I talked in detail about creating a multilingual ASP.NET MVC 3 website and showed the different techniques that can be used to globalize an MVC 3 website. Perhaps the best way to learn about internationalization is by applying it to a real-world application. In this post, I am going to show you how to globalize the popular web application, NerdDinner. We’ll tackle the obstacles one by one and from server-side to client-side. We will add the Spanish language to the website, so we can have Spanish nerds joining us. 

### Introduction

If a website targets users from different parts of the world, these users might like to see the website content in their own language. Creating a multilingual website is not an easy task, but it will certainly allow a web site to reach more audience. Fortunately, the .NET Framework already has components that support different languages and cultures.

We will revamp the NerdDinner web application, so that:

*    It can display contents in different languages.
*    It autodetects the language from the user's browser.
*    It allows the user to override the language of their browser.


### Internationalization, Globalization and Localization. What confusion!

It is actually confusion. The terms defined by Microsoft are different from the rest of the industry.

In Microsoft terms, Internationalization involves Globalization and Localization. Globalization is the process of designing applications that support different cultures. Localization is the process of customizing an application for a given culture. Other industries, however, transpose the meanings of Internationalization and Globalization. You can read more about this [here](http://en.wikipedia.org/wiki/Internationalization_and_localization).

### Internationalization or Globalization?

In this post, it makes sense to use Microsoft’s terminology since both .NET and ASP.NET MVC 3 are Microsoft’s products.

Anyway, Internationalization is often abbreviated to "I18N". The abbreviation takes the first and last letters and the number of letters between them, so 18 stands for the number of letters between the first "I" and the last "N". The same applies to Globalization (G11N), and Localization (L10N).

A culture is a language and, optionally, a region. The format for the culture name is `<languagecode2>-<country/regioncode2>`, where `<languagecode2>` is the language code and `<country/regioncode2>` is the subculture code. Examples include "es-CL" for Spanish (Chile) and "en-US" for English (United States). A complete list can be found here. When a culture has a specified language, but not a region, we call it a neutral culture. A neutral culture is not specific to any region. For example, "en" represents English, but not English of the United States.

 Now, let’s review the terms used so far:

 *   Globalization (G11N): The process of making an application support different languages and regions.
*    Localization (L10N): The process of customizing an application for a given language and region.
*    Internationalization (I18N): Describes both globalization and localization.
*    Culture: It is a language and, optionally, a region.
*    Locale: A locale is the same as a culture.
*    Neutral culture: A culture that has a specified language, but not a region. (e.g. "en", "es")
*    Specific culture: A culture that has a specified language and region. (e.g. "en-US", "en-GB", "es-CL")

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
$ 5.600,00 // Currency in es-CL, Spanish (Chile)

26/07/2011 // Date in es-MX, Spanish (Mexico)
$5,600.00 // Currency in es-MX, Spanish (Mexico)
```

You can notice the difference in date and currency format. The decimal separator in each region is different and can confuse people in the other region. If a Mexican user types one thousand in their culture "1,000", it will be interpreted as 1 (one) in a Chilean culture website. We mainly need regions for this type of reasons and not much for the language itself. 

### Culture and UICulture

ASP.NET keeps track of two culture values, the Culture and UICulture. The culture value determines the results of culture-dependent functions, such as the date, number, and currency formatting. The UICulture determines which resources are to be loaded for the page by the ResourceManager. The ResourceManager simply looks up culture-specific resources that is determined by CurrentUICulture. Every thread in .NET has CurrentCulture and CurrentUICulture objects. So ASP.NET inspects these values when rendering culture-dependent functions. For example, if current thread's culture (CurrentCulture) is set to "en-US" (English, United States), DateTime.Now.ToLongDateString() shows "Saturday, January 08, 2011", but if CurrentCulture is set to "es-CL" (Spanish, Chile) the result will be "sábado, 08 de enero de 2011".

### How can ASP.NET guess the user's language?

On each HTTP request, there is a header field called Accept-Language which determines which languages the user’s browser supports:

```
Accept-Language: en-us,en;q=0.5
```

This means that my browser prefers English (United States), but it can accept other types of English. The "q" parameter indicates an estimate of the user’s preference for that language. You can control the list of languages using your web browser.

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-1.png)
#### Internet Explorer 

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-2.png)
#### Firefox

### How to Support Different Cultures in ASP.NET MVC 3
There are two ways to incorporate different cultures in ASP.NET MVC 3:

*    By using resource strings in all our site views.
*    By using different set of views for every culture.
*    By mixing between 1 and 2

### Which one is the best?

It is a matter of convenience. Some people prefer to use a single view for all languages because it is more maintainable. While others think replacing views content with code like "@Resources.Something" might clutter the views and will become unreadable. Some project requirements force developers to implement different views per language. But sometimes you have no choice where layout has to be different like right-to-left languages. Even if you set `dir="rtl"`, this won't be enough in real applications unless the project's UI layout is really simple. Perhaps, a mix of the two is the best. Anyway, for this example, it makes sense to use resources since we won't have any issue with the layout for the Spanish and English languages that we will use.

### Internationalizing NerdDinner

Download the NerdDinner project from [CodePlex](http://nerddinner.codeplex.com/SourceControl/list/changesets) site and extract the VS2010-MVC3-Razor folder. We will do the following steps in order to convert NerdDinner into a multilingual website:

 1.   Resources: We’ll create resources files for each culture.
1.    Views: All text in the views must be extracted and added in the resource files instead.
1.    Source code: All text in C# code files must be extracted and added in the resource files, too.
1.    Model and Validation Rules: Validation rules messages have to be moved to Resources.
1.    Create a Culture Helper and a Base Controller.
1.    Client-Side: All client side code must be localized including dates, numbers, etc.

### Resource files

We will store resource files in a separate assembly, so we can reference them in other project types in the future.

Right click on the Solution and then choose the "Add->New Project" context menu command. Choose "Class Library" project type and name it "Resources".

Now right click on "Resources" project and then choose "Add->New Item" context menu command. Choose "Resource File" and name it "Resources.resx". This will be our default culture (en-US) since it has no special endings. A default culture means that when the user asks for a language that we did not implement, they will see the English version. 

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-3.png)

We want to enable Spanish, but which Spanish? Or rather, which region?

We certainly want to have nerds from all Spanish-speaking countries including Argentina and Peru, and not only from a specific country. In this case, we care about the language, not the region, so we don’t care about the country in the translation strings. We don’t need to create a resource file for every culture "es-CL", "es-MX", etc. We just need one common file.

Now create a new resource file and name it "Resources.es.resx" for the Spanish version of the site.

Before we move one, make sure to mark the resource's access modifier property to "public" in both files, so it will be accessible from other projects.

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-4.png)

### Views

We need to extract the English text from all the views and move it to the resource files. This is going to take a while as we have to do this for every single view. You can download the whole NerdDinner project internationalized from the link at the top of this page.

Now we need to reference "Resources" project from our web application, so that we can read the resource strings right from our web site. Right click on "References" under our web project "NerdDinner", and choose the "Resources" project from Projects tab.

Here is a trick, instead of typing the namespace name each time (e.g. "@Resources. Resources.LogOn "), we can add this namespace to the views Web.config and type "@Resources.LogOn" instead. Open the Web.config file under the views folder.

You also get IntelliSense when accessing these resource strings since Visual Studio generates a class behind the scenes that contains a typed property for each key. Now we need to populate the resource files. Visual Studio has a nice resource editor that makes it easy to manage our translated strings.

Let "_LoginStatus.cshtml" be our first view. We need to extract "Log On", "Log Off", and "Welcome". That’s all what we need from this view.

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-5.png)
#### English

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-6.png)
#### Spanish

We have to repeat these steps for all other views in the project. Remember that link names must be translated like in this case. Everything that appears to the end user must be translated, but element names, ids should never be translated. In this project, I made the effort to do the translation myself, but normally you as a developer will receive all the text already translated.

Note: There is no need to show all views in this post since they are done in a very similar way.

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-7.png)


### Validation Rules Internationalization

It is not surprising that controllers and other source code files contain text that needs to be translated, such as error messages sent to the end user. In order to globalize validation messages, we need to specify a few extra parameters. The "ErrorMessageResourceType" indicates the type of resource to look up the error message. "ErrorMessageResourceName" indicates the resource name to lookup the error message. Resource manager will pick the correct resource file based on the current culture.

```csharp
namespace NerdDinner.Models
{
    public class LogOnModel
    {
        [Required(ErrorMessageResourceName = "YouMustSpecifyUsername", ErrorMessageResourceType = typeof(Resources.Resources))]
        [Display(Name = "ModelUsername", ResourceType=typeof(Resources.Resources)) ]        
        public string UserName { get; set; }

        [Required(ErrorMessageResourceName = "YouMustSpecifyPassword", ErrorMessageResourceType = typeof(Resources.Resources))]
        [DataType(DataType.Password)]
        [Display(Name = "ModelPassword", ResourceType = typeof(Resources.Resources))]
        public string Password { get; set; }

        [Display(Name = "ModelRememberMe", ResourceType = typeof(Resources.Resources))]
        public bool RememberMe { get; set; }
    }

    public class RegisterModel
    {
        [Required(ErrorMessageResourceName = "YouMustSpecifyUsername", ErrorMessageResourceType = typeof(Resources.Resources))]
        [Display(Name = "ModelUsername", ResourceType = typeof(Resources.Resources))]   
        public string UserName { get; set; }

        [Required(ErrorMessageResourceName = "YouMustSpecifyEmailAddress", ErrorMessageResourceType = typeof(Resources.Resources))]
        [DataType(DataType.EmailAddress)]
        [Display(Name = "ModelEmail", ResourceType=typeof(Resources.Resources)) ]                
        public string Email { get; set; }

        [Required(ErrorMessageResourceName = "YouMustSpecifyPassword", ErrorMessageResourceType = typeof(Resources.Resources))]
        [StringLength(100, ErrorMessageResourceName = "PasswordTooShort", ErrorMessageResourceType = typeof(Resources.Resources), MinimumLength = 6)]
        [DataType(DataType.Password)]
        [Display(Name = "ModelPassword", ResourceType = typeof(Resources.Resources))]
        public string Password { get; set; }

        [DataType(DataType.Password)]
        [Display(Name = "ModelConfirmPassword", ResourceType = typeof(Resources.Resources))]
        [Compare("Password", ErrorMessageResourceName = "NewPasswordAndConfirmationMismatch", ErrorMessageResourceType = typeof(Resources.Resources))]
        public string ConfirmPassword { get; set; }
    }

}
```

### Culture Helper and Base Controller

Before we move to Client-side, we need to figure out how to retrieve the culture from the user’s browser. I already mentioned there is a header field called "Accept-Language" that the browser sends on every request. This field contains a list of culture names (language-country) that the user has configured in their browser. The problem is that this culture may not reflect the real user's preferred language, such as a computer in a cybercafé. We should allow the user to choose a language explicitly and allow them to change it. In order to do this sort of things, we need to store the user's preferred language in a store, which can be perfectly a cookie.

We will create a base controller that inspects the cookie contents first, if there is no cookie, we will use the "Accept-Language" field sent by their browser. Create a controller and name it "BaseController" like below:

```csharp
namespace NerdDinner.Controllers
{
    public class BaseController : Controller
    {
        protected override void ExecuteCore()
        {
            string cultureName = null;
            // Attempt to read the culture cookie from Request
            HttpCookie cultureCookie = Request.Cookies["_culture"];
            if (cultureCookie != null)
                cultureName = cultureCookie.Value;
            else
                cultureName = Request.UserLanguages[0]; // obtain it from HTTP header AcceptLanguages

            // Validate culture name
            cultureName = CultureHelper.GetImplementedCulture(cultureName); // This is safe


            // Modify current thread's cultures            
            Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo(cultureName);
            Thread.CurrentThread.CurrentUICulture = Thread.CurrentThread.CurrentCulture;

            base.ExecuteCore();
        }

    }
}
```

Make sure all controllers in this project inherit from this BaseController.

The base controller checks if the cookie exists, and sets the current thread cultures to that cookie value. Of course, because cookie content can be manipulated on the client side, we should always validate its value using a helper class called "CultureHelper". If the culture name is not valid, the helper class returns the default culture. 

### CultureHelper Class

The CultureHelper is basically a utility that allows us to store culture names we are implementing in the web site:


![](/images/posts/archived/aspnet-mvc-internationalization-part-2-8.png)

```csharp
namespace NerdDinner.Helpers
{
    public static class CultureHelper
    {
        // Valid cultures
        private static readonly IList<string> _validCultures = new List<string> {"af", "af-ZA", "sq", "sq-AL", "gsw-FR", "am-ET", "ar", "ar-DZ", "ar-BH", "ar-EG", "ar-IQ", "ar-JO", "ar-KW", "ar-LB", "ar-LY", "ar-MA", "ar-OM", "ar-QA", "ar-SA", "ar-SY", "ar-TN", "ar-AE", "ar-YE", "hy", "hy-AM", "as-IN", "az", "az-Cyrl-AZ", "az-Latn-AZ", "ba-RU", "eu", "eu-ES", "be", "be-BY", "bn-BD", "bn-IN", "bs-Cyrl-BA", "bs-Latn-BA", "br-FR", "bg", "bg-BG", "ca", "ca-ES", "zh-HK", "zh-MO", "zh-CN", "zh-Hans", "zh-SG", "zh-TW", "zh-Hant", "co-FR", "hr", "hr-HR", "hr-BA", "cs", "cs-CZ", "da", "da-DK", "prs-AF", "div", "div-MV", "nl", "nl-BE", "nl-NL", "en", "en-AU", "en-BZ", "en-CA", "en-029", "en-IN", "en-IE", "en-JM", "en-MY", "en-NZ", "en-PH", "en-SG", "en-ZA", "en-TT", "en-GB", "en-US", "en-ZW", "et", "et-EE", "fo", "fo-FO", "fil-PH", "fi", "fi-FI", "fr", "fr-BE", "fr-CA", "fr-FR", "fr-LU", "fr-MC", "fr-CH", "fy-NL", "gl", "gl-ES", "ka", "ka-GE", "de", "de-AT", "de-DE", "de-LI", "de-LU", "de-CH", "el", "el-GR", "kl-GL", "gu", "gu-IN", "ha-Latn-NG", "he", "he-IL", "hi", "hi-IN", "hu", "hu-HU", "is", "is-IS", "ig-NG", "id", "id-ID", "iu-Latn-CA", "iu-Cans-CA", "ga-IE", "xh-ZA", "zu-ZA", "it", "it-IT", "it-CH", "ja", "ja-JP", "kn", "kn-IN", "kk", "kk-KZ", "km-KH", "qut-GT", "rw-RW", "sw", "sw-KE", "kok", "kok-IN", "ko", "ko-KR", "ky", "ky-KG", "lo-LA", "lv", "lv-LV", "lt", "lt-LT", "wee-DE", "lb-LU", "mk", "mk-MK", "ms", "ms-BN", "ms-MY", "ml-IN", "mt-MT", "mi-NZ", "arn-CL", "mr", "mr-IN", "moh-CA", "mn", "mn-MN", "mn-Mong-CN", "ne-NP", "no", "nb-NO", "nn-NO", "oc-FR", "or-IN", "ps-AF", "fa", "fa-IR", "pl", "pl-PL", "pt", "pt-BR", "pt-PT", "pa", "pa-IN", "quz-BO", "quz-EC", "quz-PE", "ro", "ro-RO", "rm-CH", "ru", "ru-RU", "smn-FI", "smj-NO", "smj-SE", "se-FI", "se-NO", "se-SE", "sms-FI", "sma-NO", "sma-SE", "sa", "sa-IN", "sr", "sr-Cyrl-BA", "sr-Cyrl-SP", "sr-Latn-BA", "sr-Latn-SP", "nso-ZA", "tn-ZA", "si-LK", "sk", "sk-SK", "sl", "sl-SI", "es", "es-AR", "es-BO", "es-CL", "es-CO", "es-CR", "es-DO", "es-EC", "es-SV", "es-GT", "es-HN", "es-MX", "es-NI", "es-PA", "es-PY", "es-PE", "es-PR", "es-ES", "es-US", "es-UY", "es-VE", "sv", "sv-FI", "sv-SE", "syr", "syr-SY", "tg-Cyrl-TJ", "tzm-Latn-DZ", "ta", "ta-IN", "tt", "tt-RU", "te", "te-IN", "th", "th-TH", "bo-CN", "tr", "tr-TR", "tk-TM", "ug-CN", "uk", "uk-UA", "wen-DE", "ur", "ur-PK", "uz", "uz-Cyrl-UZ", "uz-Latn-UZ", "vi", "vi-VN", "cy-GB", "wo-SN", "sah-RU", "ii-CN", "yo-NG" };

        // Include ONLY cultures you are implementing
        private static readonly IList<string> _cultures = new List<string> {
            "en-US",  // first culture is the DEFAULT
            "es", // Spanish NEUTRAL culture
            "es-AR", "es-BO", "es-CL", "es-CO", "es-CR", "es-DO", "es-EC", "es-SV", "es-GT", "es-HN", "es-MX", "es-NI", "es-PA", "es-PY", "es-PE", "es-PR", "es-ES", "es-US", "es-UY", "es-VE" // Specific cultures
           
        };



        /// <summary>
        /// Returns a valid culture name based on "name" parameter. If "name" is not valid, it returns the default culture "en-US"
        /// </summary>
        /// <param name="name">Culture's name (e.g. en-US)</param>
        public static string GetImplementedCulture(string name)
        {
            // make sure it's not null
            if (string.IsNullOrEmpty(name))
                return GetDefaultCulture(); // return Default culture

            // make sure it is a valid culture first
            if(_validCultures.Where( c => c.Equals(name, StringComparison.InvariantCultureIgnoreCase)).Count() == 0)
                return GetDefaultCulture(); // return Default culture if it is invalid

               
            // if it is implemented, accept it
            if (_cultures.Where( c => c.Equals(name, StringComparison.InvariantCultureIgnoreCase)).Count() > 0)
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
            if (name.Length < 2)
                return name;

            return name.Substring(0, 2); // Read first two chars only. E.g. "en", "es"
        }

 
    


    }
}
```

We should populate "_cultures" manually. The "_cultures" dictionary stores the list of culture names our site supports. The nice part of this utility class is that it serves similar cultures. For example, if a user is visiting our site from the United Kingdom (en-GB), a culture which is not implemented in our site, he or she will see our site in English using "en-US" (English, United States). This way, you don't have to implement all cultures unless you really care about currency, date format, etc. One important thing to mention is that we added all specific cultures for the Spanish language. For example, we are implementing "es" Spanish in general without any region where a date looks like this "30/07/2011". Now suppose a nerd is coming from Chile (es-CL), it would be very nice display dates in their culture format (30-07-2011) instead of the neutral one. It does not matter if we don’t have a resource file for any of these cultures. ResourceManager can pick a neutral culture when it cannot find a specific culture file, this automatic mechanism is called fallback.

### Output Caching

You might run into a problem with Output Caching as content varies by culture. If you think that VaryByHeader="Accept-Language" could do it, then this is not always the case because our culture is stored in a cookie, not in the header. If the cookie is absent, this approach will be OK. We overcome this by applying custom caching. 


```csharp
   /* NerdDinner i18n Custom caching */
   public override string GetVaryByCustomString(HttpContext context, string arg)
        {
            // It seems this executes multiple times and early, so we need to extract language again from cookie.
            if (arg == "culture") // culture name (e.g. "en-US") is what should vary caching
            {
                string cultureName = null;
                // Attempt to read the culture cookie from Request
                HttpCookie cultureCookie = Request.Cookies["_culture"];
                if (cultureCookie != null)
                    cultureName = cultureCookie.Value;
                else
                    cultureName = Request.UserLanguages[0]; // obtain it from HTTP header AcceptLanguages

                // Validate culture name
                cultureName = CultureHelper.GetImplementedCulture(cultureName); // This is safe

                return cultureName.ToLower();// use culture name as cache key, "es", "en-us", "es-cl", etc.
            }
            
            return base.GetVaryByCustomString(context, arg);
        }
```

And it can be used this way:

```csharp
[OutputCache(Duration=60, VaryByCustom="culture")]
```

### Client Side

This is the last step in the process of i18n. Client side internationalization can be easy or challenging depending on the way you created your code and the tools you are using. The first thing to do is to replace all the strings inside javascript files with the translated versions. Then we need to figure out how to parse and format numbers, dates, and currency if any. We also need to localize the calendar used to set event dates.

Before we begin with the strings replacement, it is worth it to mention [JQuery Global plugin](https://github.com/jquery/jquery-global). This jQuery plugin supports i18n for hundreds of cultures. It makes it easy to parse and format numbers, dates, and currencies. We will use this plugin in the NerdDinner project. Download the plugin and extract the folder "lib" contents in the "Scripts" folder of our project. 

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-9.png)

Add a reference to the "globalize.js" file in the "_Layout.cshtml". The jQuery Global plugin allows us to store culture specific values, so this can be the perfect place to store the translated strings. Of course, there are other ways to store localized strings inside javascript files. For example, we can create a separate javascript file for each culture, so that each file contains localized strings for that culture. However, it is simpler to store strings using this plugin.

Let's begin with NerdDinner.js file. Look at the following code snippet:

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-10.png)

This method is fragile. It is appending an "s" to the end of the string. For English, this is fine. However, this won’t work in many other cultures, so we need two separate words "RSVP" and "RSVPs" stored separately. We can store these words in resource files and fetch them from there with this trick. There is also a confirmation message we need to replace. Here is how the code will look like:

```csharp
  Globalize.addCultureInfo("@NerdDinner.Helpers.CultureHelper.GetCurrentCulture()" /*es-CL*/, {
                    messages: {
                        "RSVP": "@Resources.RSVP",
                        "RSVPs": "@Resources.RSVPs",
                        "BingMapsReturnedAddress": "@Html.Raw(Resources.BingMapsReturnedAddress)",
                        "MonthDayYear": "@Html.Raw(Resources.MonthDayYear)",
                        "MonthDay": "@Html.Raw(Resources.MMMdd)",
                        "with":"@Resources.with"
                    }
                });
```

And now `NerdDinner.js` should look like below:

```javascript
function _getRSVPMessage(RSVPCount)  {
    var rsvpMessage  =  ""  +  RSVPCount  +  " "  +  (RSVPCount  >  1  ?  Globalize.localize("RSVPs")  :  Globalize.localize("RSVP")); 
    return rsvpMessage; 
}    
```

And here is how you would replace the confirmation:

```javascript
var answer  =  confirm(htmlDecode(Globalize.localize("BingMapsReturnedAddress").replace("{0}",  locations[0].Name).replace("{1}",  currentAddress.val())));
```

Now let’s move on to dates. Currently, the NerdDinner project is using a different jQuery plugin for date formatting. We need to replace it with the Global plugin.

Look at the following function:

```javascript
function _getDinnerDate(dinner,  formatStr)  {
        return '<strong>'  +  _dateDeserialize(dinner.EventDate).format(formatStr)  +  '</strong>'; 
}
```

Here is how it should be after using Global plugin:

```javascript
      function _getDinnerDate(dinner,  formatStr)  {
                  return '<strong>'  +  Globalize.format(_dateDeserialize(dinner.EventDate),  formatStr); 
     + '</strong>'; 
       }        
```

### The Calendar

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-11.png)

There are two components that need to be localized: DatePicker and TimePicker. Luckily, both support i18n. DatePicker ships with javascript localization files, so we just need to reference the file of the implemented culture. The following references the right file from Microsoft CDN:

```html
<script src="http://ajax.aspnetcdn.com/ajax/jquery.ui/1.8.11/i18n/jquery.ui.datepicker-@(NerdDinner.Helpers.CultureHelper.GetCurrentNeutralCulture()).js" type="text/javascript"></script>
```

It is important to mention that the date picker plugin only supports neutral cultures (e.g. "en", "es"). This means we need to tweak its formatting.

The time picker plugin localization is done in the same way, but it doesn’t ship with its own localization file, so we need to do the process manually. Here is the complete js code for both components:

```javascript
  @if (NerdDinner.Helpers.CultureHelper.GetCurrentCulture() != "en" && NerdDinner.Helpers.CultureHelper.GetCurrentCulture() != "en-US")
    {
        // load necessary localization files for Global and DatePicker
         <script src="http://ajax.aspnetcdn.com/ajax/jquery.ui/1.8.11/i18n/jquery.ui.datepicker-@(NerdDinner.Helpers.CultureHelper.GetCurrentNeutralCulture()).js" type="text/javascript"></script>
        
         <script type="text/javascript">
             $(function () {
                 
                 @* Unfortunately, the datepicker only supports Neutral cultures, so we need to adjust date and time format to the specific culture *@
                 $("#EventDate").change(function(){
                     $(this).val(Globalize.format($(this).datetimepicker('getDate'), Globalize.culture().calendar.patterns.d + " " + Globalize.culture().calendar.patterns.t)); /*d t*/
                });
                
                @* Timepicker i18n *@
                $.timepicker.regional['@NerdDinner.Helpers.CultureHelper.GetCurrentCulture()'] = {
                timeOnlyTitle: '@Resources.timeOnlyTitle',
                timeText: '@Resources.timeText',
                hourText: '@Resources.hourText',
                minuteText: '@Resources.minuteText',
                secondText: '@Resources.secondText',
                currentText: '@Resources.currentText',
                closeText: '@Resources.closeText',
                ampm: '@Resources.ampm'
            };

            $.timepicker.setDefaults($.timepicker.regional['@NerdDinner.Helpers.CultureHelper.GetCurrentCulture()']);

             });
         </script>
    }
```

DatePicker doesn’t support specific cultures like "es-CL". This will be an issue for ASP.NET. For example, the date value "30/07/2011" is not a valid date in the Chilean culture, which uses "-" (i.e. "30-07-2011"). ASP.NET will throw an exception when parsing a date in an incorrect format. To solve this issue, each time the user modifies the event date, we need to re-format it using Globalize.format method.

For TimePicker, we are providing the plugin the translated strings from the resource files. We are telling the plugin to use the Spanish culture like this:

```javascript
$.timepicker.setDefaults($.timepicker.regional['@NerdDinner.Helpers.CultureHelper.GetCurrentCulture()']);
```

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-12.png)

Now you can see the calendar in Spanish

Now that dates and time appear in correct format to the end user, let’s move to numbers parsing and formatting.

**NOTE**  It seems that DefaultModelBinder is not culture-ware. It does not always use the current culture for type conversion. If the data value is coming as part of the URL, the InvariantCulture takes place. Otherwise, the current culture is used. This might be a big issue. Luckily, it’s not a problem in this case since we are not using any values as part of a URL. Anyway, the invariant culture is culture-insensitive, and it is similar to English but is not related to any region. Suppose a culture on the server is set to "es-CL" which expects dates in this format "30-07-2011" and an incoming querystring has the value "01-08-2011". An exception is thrown since DefaultModelBinder used the InvariantCulture which can parse dates in this format "mm/dd/yyyy" or "yyyy-mm-dd". 

### Numbers

Although the end user is not dealing with any numbers directly, there are two hidden input fields that actually store the Bing Maps pushpin coordinates. Since these values are of type float, they contain a decimal point which poses a problem across different cultures as explained earlier. To overcome this issue, each time these values are read or saved, we need to parse or format them. Otherwise, we’ll get a server-side error from ASP.NET. Anyway, we can use Globalize.parseFloat and Globalize.format for this purpose. Below is some code snippet from NerdDinner.js

```javascript
 if (NerdDinner._points.length  ===  1)  {
           $("#Latitude").val(Globalize.format(NerdDinner._points[0].Latitude,  "n14")); 
           $("#Longitude").val(Globalize.format(NerdDinner._points[0].Longitude,  "n14")); 
       }
```

### Unobtrusive Client-Side Validation

Client side validation is important in nowadays web applications because users expect immediate response. It’s not a good thing to make the a user wait for the server load response just to tell them if "10" is a valid number. A lot of things can be done on the client side which can also boost performance by reducing the number of sever requests. Unobtrusive validation does not add any javascript in the views. Instead, the validation process is done separately. However, validation rules are described as attributes that are added to the HTML elements to be validated. These attributes are processed by some MVC client side library which configures jQuery Validation tool that does all the job behind the scenes.

Anyway, NerdDinner is using unobtrusive client-side validation, and jQuery validation by default does not consider i18n. We need to override some of the validation methods in order to have i18n. In our NerdDinner case, the only problem occurs when we try to place an event since the Latitude and Longitude input values can be misinterpreted in different cultures.

We can either turn off client-side validation for these inputs, or we can tell jQuery validation library to parse them correctly. We’ll choose the latter since in other applications, it may not be pleasant to turn off validation. 

```javascript
// Override default jQuery Validation number method
        $.validator.methods.number  =   function(value,  element)  {
               return  this.optional(element)  ||  !isNaN(Globalize.parseFloat(value)); 
             }
```

In this method, the return value is either true or false. We are checking if the result of Globalize.parseFloat is a valid number. 

### Language picker

Finally, we need to allow the user to choose their preferred language. We need a language selector. I already created a one for this purpose. Below is the code needed to add the language picker to the _Layout.cshtml view:

```html
            <div class="lang-picker-wrapper">
            <span class="lang-picker">
                 <a class="en-us" href="javascript:void(0);">English</a>
                 <a class="@(NerdDinner.Helpers.CultureHelper.GetNeutralCulture(Request.UserLanguages[0].ToLower()) == "es" ? Request.UserLanguages[0].ToLower() : "es")" href="javascript:void(0);">Español</a>
            </span>
            </div>           
```

We used the AcceptLanguage header field here on purpose. Suppose a nerd is visiting our site from Spain, instead of using the neutral culture "es", we want to use the user’s specific culture "es-ES" instead. However, if the user’s browser does not prefer Spanish and they prefer to see our site in Spanish, the neutral culture is applied.

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-13.png)

```javascript
 $(".lang-picker a." + Globalize.culture().name.toLowerCase()).prependTo($(".lang-picker")); @* Select current culture*@

                @* culture change *@
                $(".lang-picker a").click(function(){

                if ($(this).hasClass(Globalize.culture().name.toLowerCase()))
                    return false; // do nothing

                $.cookie("_culture", $(this).attr("class") , {expires : 365, path: '/'});
                window.location.reload(); // reload 

                });
```
And the javascript needed for this to work:

In the code above, we attach a click handler to each link, so the user can click on them to change their language. The culture name is stored in the "class" attribute of the link. When the language link is clicked, the cookie’s value is updated, and the page is reloaded to reflect the new culture.

### Try It Out

Let's see how NerdDinner looks like now:

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-14.png)

Everything looks fine except for one thing, the image next to the search text box is still in English because it is not text. We have two options: Either we convert this image into text, or we create a different image file per language. I have already prepared a Spanish version of this image. It doesn’t look exactly the same, but it’s still good. We have to reference the right image file in CSS. However, we can’t use C# directly in CSS files, so we need to move the style into our view.

```css
    <style type="text/css">
        #hm-masthead
        {
            background: transparent url('Content/img/hm-masthead-@(NerdDinner.Helpers.CultureHelper.GetCurrentNeutralCulture()).png') no-repeat 0px 45px;
        }
    
    </style>
```

We used the neutral version of the culture because we only need one image per language, not per region.

Enjoy.

Now we can have Spanish nerds joining us!! 

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-15.png)

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-16.png)

![](/images/posts/archived/aspnet-mvc-internationalization-part-2-17.png)

### Summary

Building a multilingual web application is not an easy task, but it's worth it especially for web applications targeting users from all over the world, something which many sites do. It is true that i18n is not the first priority in site development process; however, it should be well planned early in the stage of development, so it can be easily implemented in the future. Luckily, ASP.NET supports i18n and there are plenty of .NET classes that are handy.

Client-side localization was more painful than server-side since there were many components involved, and we had to localize them one by one. Anyway, if i18n is planned from the beginning, it can mitigate the pain to convert the web project into a multilingual website.

I hope his helps.
Any questions or comments are welcome.