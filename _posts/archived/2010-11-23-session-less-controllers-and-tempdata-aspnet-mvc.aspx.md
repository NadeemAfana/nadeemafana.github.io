---
layout: post
title: Create a Wizard in ASP.NET MVC 3
redirect_from:
- "/post/session-less-controllers-and-TempData-ASPNET-MVC.html"
- "/post/session-less-controllers-and-TempData-ASPNET-MVC/"
tags: [aspnetmvc, aspnet, mvc, session]
---
**[Download Code](/attachments/posts/archived/session-less-controllers-and-tempdata-aspnet-mvc.zip)**

### Introduction

Microsoft has released ASP.NET MVC 3 RC with a dozen of exciting features. One of these features is Sessionless controllers, meaning you can specify the type of session state behavior that is required in order to support HTTP requests to a controller. 

### Sessionless Controller

If you are not using Session State in your web site, you can gain slight performance improvement by disabling session state globally. However, if you use session state in some controllers, you can keep it in those controllers, and disable it in those that are not using it.

The new ControllerSessionStateAttribute gives you more control over session-state behavior for controllers by specifying a System.Web.SessionState.SessionStateBehavior enumeration value. The following example shows how to turn off session state for all requests to a controller.

```csharp
[ControllerSessionState(SessionStateBehavior.Disabled)]
public class HomeController : Controller {
  public ActionResult Index() {
    
  
  }
}
```

 The [SessionStateBehavior](http://msdn.microsoft.com/en-us/library/system.web.sessionstate.sessionstatebehavior.aspx) Enumeration Specifies the type of session support which is one of the following values:

*    Disabled: Session state is disabled for the processing request.
*    ReadOnly: Read-only session state is enabled for the request. This means that session state cannot be updated.
*    Required: Full read-write session state behavior is enabled for the request
*    Default: The default ASP.NET logic is used to determine the session state behavior for the request.

Recall that TempData uses Session State behind the scenes, and using it in any controller that has session state disabled will throw an exception. However, it is still possible to use TempData in that case by configuring TempData to use browser cookies instead of Session State. Below is revamped code to use a browser cookie as a storage location instead of Session State. 

First, you create a base class controller from which your controllers derive: 

```csharp
 public class ControllerBase : Controller
    {
        // Store TempData in browser's cookie instead
        protected override ITempDataProvider CreateTempDataProvider()
        {
            return new CookieTempDataProvider(HttpContext);
        }
        
    
    }
```

Just derive all your classes from BaseController. Here is the `CookieTempDataProvider`: 

```csharp
 public class CookieTempDataProvider : ITempDataProvider
    {
        internal const string TempDataCookieKey = "__ControllerTempData";
        HttpContextBase _httpContext;

        public CookieTempDataProvider(HttpContextBase httpContext)
        {
            if (httpContext == null)
            {
                throw new ArgumentNullException("httpContext");
            }
            _httpContext = httpContext;
        }

        public HttpContextBase HttpContext
        {
            get
            {
                return _httpContext;
            }
        }

        protected virtual IDictionary<string, object> LoadTempData(ControllerContext controllerContext)
        {
            HttpCookie cookie = _httpContext.Request.Cookies[TempDataCookieKey];
            if (cookie != null && !string.IsNullOrEmpty(cookie.Value))
            {
                IDictionary<string, object> deserializedTempData = DeserializeTempData(cookie.Value);



                return deserializedTempData;
            }

            return new Dictionary<string, object>();
        }

        protected virtual void SaveTempData(ControllerContext controllerContext, IDictionary<string, object> values)
        {
            bool isDirty = (values != null && values.Count > 0);

          
            string cookieValue = SerializeToBase64EncodedString(values);  
            var cookie = new HttpCookie(TempDataCookieKey);
            cookie.HttpOnly = true;

            // Remove cookie
            if (!isDirty)
            {
                cookie.Expires = DateTime.Now.AddDays(-4.0);
                cookie.Value = string.Empty;

                _httpContext.Response.Cookies.Set(cookie);

                return;

            }
            cookie.Value = cookieValue;

            _httpContext.Response.Cookies.Add(cookie);
        }

        public static IDictionary<string, object> DeserializeTempData(string base64EncodedSerializedTempData)
        {
            byte[] bytes = Convert.FromBase64String(base64EncodedSerializedTempData);
            var memStream = new MemoryStream(bytes);
            var binFormatter = new BinaryFormatter();
            return binFormatter.Deserialize(memStream, null) as IDictionary<string, object> /*TempDataDictionary : This returns NULL*/;
        }

        public static string SerializeToBase64EncodedString(IDictionary<string, object> values)
        {
            MemoryStream memStream = new MemoryStream();
            memStream.Seek(0, SeekOrigin.Begin);
            var binFormatter = new BinaryFormatter();
            binFormatter.Serialize(memStream, values);
            memStream.Seek(0, SeekOrigin.Begin);
            byte[] bytes = memStream.ToArray();
            return Convert.ToBase64String(bytes);
        }

        IDictionary<string, object> ITempDataProvider.LoadTempData(ControllerContext controllerContext)
        {
            return LoadTempData(controllerContext);
        }

        void ITempDataProvider.SaveTempData(ControllerContext controllerContext, IDictionary<string, object> values)
        {
            SaveTempData(controllerContext, values);
        }
    }
```

**Do not store any sensitive data inside TempData now as it is sent to the client, and it can be easily decoded with minimal effort.**