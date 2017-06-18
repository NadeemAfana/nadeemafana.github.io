---
layout: post
title: Extension Methods Library
redirect_from:
- "/post/Extension-Methods-Library.aspx/"
tags: [C#, .net, extension methods, library]
---
Extension methods are a compiler trick which allows static methods to be called using instance methods syntax. They are so convenient that LINQ is essentially a collection of extension methods. Extension methods are a good way to shrink your code and follow the DRY principle. For example, instead of the following long code

```csharp
bool IsValidOs(string os)
{
    return os.Equals("Windows", StringComparison.OrdinalIgnoreCase) ||
           os.Equals("Linux", StringComparison.OrdinalIgnoreCase) ||
           os.Equals("Mac OS", StringComparison.OrdinalIgnoreCase);
}
```
The following version is much more readable:
```csharp
bool IsValidOs(string os)
{
    return os.IsLikeAny("Windows", "Linux", "Mac OS");
}
```
I have been using several extension methods in my code for a while, but then I decided to share them with the world.

To add the library to your project, open Package Manager Console, select your project and type

```powershell
Install-Package ExtensionMethods
```
![Install-Package ExtensionMethods](/images/posts/archived/extension-methods-library-1.png "Install-Package ExtensionMethods")

Import the namespace

```csharp
using ExtensionMethods;
```

Here is a list of the included methods:

*  [ToFormat](#ToFormat)
*  [ToFormatNamed](#ToFormatNamed)
*  [Reverse](#Reverse)
*  [Left](#Left)
*  [Right](#Right)
*  [IsLike](#IsLike)
*  [IsLikeAny](#IsLikeAny)
*  [IsLikeAll](#IsLikeAll)
*  [StartsLike](#StartsLike)
*  [EndsLike](#EndsLike)
*  [TrimStart](#TrimStart)
*  [TrimEnd](#TrimEnd)
*  [Capitalize](#Capitalize)
*  [ToTitleCase](#ToTitleCase)
*  [IsNullOrWhitespace](#IsNullOrWhitespace)
*  [IsNumeric](#IsNumeric)
*  [ToBase64](#ToBase64)
*  [FromBase64](#FromBase64)
*  [ToMd5](#ToMd5)
*  [ToBytes](#ToBytes)
*  [ToInt](#ToInt)
*  [ToIntOrDefault](#ToIntOrDefault)
*  [ToDouble](#ToDouble)
*  [ToDoubleOrDefault](#ToDoubleOrDefault)
*  [ToLink](#ToLink)
*  [ContainsAny](#ContainsAny)
*  [ContainsAll](#ContainsAll)
*  [bool.ToYesOrNo](#bool_ToYesOrNo)
*  [bytes[].ToHexString](#bytes_ToHexString)
*  [IPrincipal.GetUserNameOnly](#IPrincipal_GetUserNameOnly)
*  [IPrincipal.IsInAnyRole](#IPrincipal_IsInAnyRole)
*  [IPrincipal.IsInAllRoles](#IPrincipal_IsInAllRoles)

<a id="ToFormat"></a>
## ToFormat()

Similar to `string.Format`
```csharp
"My name is {0} {1}".ToFormat("Nadeem", "Afana");  // My name is Nadeem Afana
```
You can even use it on nullable types:
```csharp
int? value = 12700;
value.ToFormat("#,#"); // 12,700 
     
value = null;
value.ToFormat("#,#"); // null
```

<a id="ToFormatNamed"></a>
## ToFormatNamed() (Deprecated)
**Note**: Use C# 6 interpolated strings.

Similar to `string.Forma`t but uses named formats for better readability.
```csharp
string firstName = "Nadeem";
string lastName = "Afana";
string language = "C#";
"My name is {first} {last}, and I like {lang}.".ToFormatNamed(firstName, lastName, language); 
// My name is Nadeem Afana, and I like C#.
```
It also supports composite formatting:
```csharp
int kbps = 30000;
  
@"The day is {dayName:dddd} and the time is {hour:hh:mm}. 
The Internet speed is {speed:#.0} mbps.
".ToFormatNamed(DateTime.Now, DateTime.Now, kbps/1000.0);
// The day is Sunday and the time is 05:31. The Internet speed is 30.0 mbps.
```
The variable names do not have to match the arguments,but the order of arguments is important.

<a id="Reverse"></a>
## Reverse()

Reverses a string.

```csharp
"The quick brown fox jumps over the lazy dog".Reverse();
// god yzal eht revo spmuj xof nworb kciuq ehT
```

<a id="Left"></a>
## Left()
Reads the first n characters from the left of a string.
```csharp
"The quick brown fox jumps over the lazy dog".Left(9);
// The quick
```

<a id="Left"></a>
## Right()
Reads the first n characters from the right of a string.
```csharp
"The quick brown fox jumps over the lazy dog".Right(3);
// dog
```


<a id="Left"></a>
## IsLike()
Determines whether two strings have the same value ignoring Case.
```csharp
"STRING".IsLike("String"); // True
```

<a id="IsLikeAny"></a>
## IsLikeAny()
Determines whether a string matches any value in a collection of strings ignoring case.
```csharp
"c#".IsLikeAny("C++", "C#", "VB");  // True
```


<a id="IsLikeAll"></a>
## IsLikeAll()
Determines whether a string matches all the values in a collection of strings ignoring case.
```csharp
"internet".IsLikeAll("Internet", "INTERNET", "iNTerNet");  // True
```


<a id="StartsLike"></a>
## StartsLike()
Determines whether the beginning of a string matches the specified string ignoring case.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.StartsLike("the"); // True
```

<a id="EndsLike"></a>
## EndsLike()
Determines whether the end of a string matches the specified string ignoring case.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.EndsLike("DOG"); // True
```

<a id="TrimStart"></a>
## TrimStart()
Removes a string from the beginning of another string.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.TrimStart("The"); //  quick brown fox jumps over the lazy dog
 
s.TrimStart("THE", false); // case insensitive
//  quick brown fox jumps over the lazy dog
```

<a id="TrimEnd"></a>
## TrimEnd()
Removes a string from the end of another string.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.TrimEnd("dog"); 
// The quick brown fox jumps over the lazy 
s.TrimEnd("DOG", false); // case insensitive
// The quick brown fox jumps over the lazy 
```

<a id="Capitalize"></a>
## Capitalize()
Capitalizes the first letter of each word. Synonym for `ToTitleCase()`
```csharp
string s = "the internet";
s.Capitalize(); // The Internet
```

<a id="ToTitleCase"></a>
## ToTitleCase()
Capitalizes the first letter of each word. Synonym for Capitalize()
```csharp
string s = "the internet";
s.ToTitleCase(); // The Internet
```


<a id="IsNullOrWhiteSpace"></a>
## IsNullOrWhiteSpace()
Indicates whether a specified string is null, empty, or consists only of white-space characters.
```csharp
string s = "       ";
s.IsNullOrWhiteSpace(); // True
 
s = null;
s.IsNullOrWhiteSpace(); // True
 
s = string.Empty;
s.IsNullOrWhiteSpace(); // True
```


<a id="IsNumeric"></a>
## IsNumeric()
Determines if a string can be parsed into a number.
```csharp
"1253".IsNumeric(); // True
 
"Abc".IsNumeric(); // False
```

<a id="ToBase64"></a>
## ToBase64()
Converts a string into Base64 encoding.
```csharp
string s = "Hello World!";
s.ToBase64(); // SGVsbG8gV29ybGQh   (Utf8)
 
s.ToBase64(Encoding.UTF32); // Utf32 
// SAAAAGUAAABsAAAAbAAAAG8AAAAgAAAAVwAAAG8AAAByAAAAbAAAAGQAAAAhAAAA
```

<a id="FromBase64"></a>
## FromBase64()
Decodes a Base64-encoded string.
```csharp
"SGVsbG8gV29ybGQh".FromBase64(); // Hello World!   (Utf8)
 
"SAAAAGUAAABsAAAAbAAAAG8AAAAgAAAAVwAAAG8AAAByAAAAbAAAAGQAAAAhAAAA".FromBase64(Encoding.UTF32); // Hello World! (Utf32) 
```

<a id="ToMd5"></a>
## ToMd5()
Computes MD5 hash of a string.
```csharp
"Hello World!".ToMd5(); // ed076287532e86365e841e92bfc50d8c (Utf8)
 
"Hello World!".ToMd5(Encoding.UTF32); // b090a014200184a82c6fff39816cb5bc (Utf32)
```

<a id="ToBytes"></a>
## ToBytes()
Converts a string into an array of bytes.
```csharp
"Hello World!".ToBytes(); // Utf8
// { 72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 33};
 
"Hello World!".ToBytes(Encoding.UTF7);  // Utf7
// { 72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 43, 65, 67, 69, 45};
```

<a id="ToInt"></a>
## ToInt()
Parses an integer from a string.
```csharp
"1500".ToInt(); // 1500
```

<a id="ToIntOrDefault"></a>
## ToIntOrDefault()
Parses an integer from a string. If parsing fails, returns a default integer.
```csharp
"1500".ToIntOrDefault(); // 1500
"Abc".ToIntOrDefault(0); // 0
```

<a id="ToDouble"></a>
## ToDouble()
Parses a double from a string.
```csharp
"3.14".ToDouble(); // 3.14
```

<a id="ToDoubleOrDefault"></a>
## ToDoubleOrDefault()
Parses a double from a string. If parsing fails, returns a default double value.
```csharp
"3.14".ToDoubleOrDefault(); // 3.14
"Abc".ToDoubleOrDefault(0.0); // 0.0
```

<a id="ToLink"></a>
## ToLink()
Scans a string for valid http Urls and converts them to Html links.
```csharp
"My website is www.afana.me".ToLink();  
// My website is <a href="http://www.afana.me/">www.afana.me</a>
 
"My website is www.afana.me".ToLink("click here"); 
// My website is <a href="http://www.afana.me/">click here</a>
```

<a id="ContainsAny"></a>
## ContainsAny()
Determines whether a string contains at least one of the specified strings.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.ContainsAny("a", "fox"); // True
```


<a id="ContainsAll"></a>
## ContainsAll()
Determines whether a string contains all the specified strings.
```csharp
string s = "The quick brown fox jumps over the lazy dog";
s.ContainsAll("dog", "fox"); // True
```

<a id="bool_ToYesOrNo"></a>
## bool.ToYesOrNo()
Returns Yes or No string if a boolean value is true or false, respectively. This can be very useful especially in MVC views.
```csharp
bool isOpen = true;
"Is Open: {0}".ToFormat(isOpen.ToYesOrNo()); // Is Open: Yes
```


<a id="bytes_ToHexString"></a>
## bytes[].ToHexString()
Converts a collection of bytes into hexadecimal string respresentation.
```csharp
byte[] bytes = new byte[] { 72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 33 };
bytes.ToHexString(); // 48656c6c6f20576f726c6421
```

<a id="IPrincipal_GetUserNameOnly"></a>
## IPrincipal.GetUserNameOnly()
Returns the user name without a domain name.
```csharp
Thread.CurrentPrincipal.GetUserNameOnly(); // Domain\Nadeem => Nadeem 
```

<a id="IPrincipal_IsInAnyRole"></a>
## IPrincipal.IsInAnyRole()
Determines if a IPrincipal belongs to at least one of the specified roles.
```csharp
var user = Thread.CurrentPrincipal; 
user.IsInAnyRole("admin", "user");
```

<a id="IPrincipal_IsInAllRoles"></a>
## IPrincipal.IsInAllRoles()
Determines if a IPrincipal belongs to all the specified roles.
```csharp
var user = Thread.CurrentPrincipal; 
user.IsInAllRoles("admin", "db_admin");
```






