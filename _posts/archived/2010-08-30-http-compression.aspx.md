---
layout: post
title: HTTP Compression
redirect_from:
- "/post/http-compression.aspx.html"
- "/post/http-compression.aspx/"
tags: [aspnetmvc, aspnet, mvc, compression, iis, gzip, defalte]
---
**[Download Code](https://github.com/NadeemAfana/Session-Tracker-plugin)**

### Introduction
Reducing the amount of data sent to the client can boost performance of your website, as the time needed to send information to the client’s browser will be much shorter. This means your pages will load faster and the client will be happier. In some cases, network bandwidth is a concern, so it is worth considering HTTP compression. HTTP compression will shrink data before IIS sends it to the client’s browser; the browser then will decompress the incoming data received from IIS. HTTP compression is supported by HTTP 1.1. Most current browsers support it and have this feature enabled by default.

IIS 6.0 and later support HTTP compression, if you are running IIS 7.x, you may not need to take any additional steps to enable this feature on your site since it is enabled be default.

Browsers that support HTTP compression will send Accept-Encoding header with each request to indicate compression schemes they can handle. For example, Internet Explorer 7 sends the following HTTP header:

```html
Accept-Encoding: gzip, deflate
```

This means Internet Explorer supports both gzip and deflate compression formats.

### HTTP Compression in IIS

IIS supports two types of HTTP Compression:

1.    Static files. (ON by default)
2.    Dynamic application responses. (OFF by default)

IIS performs dynamic compression each time a client requests the content, but the compressed version is not cached to disk. This change is made because of the primary difference between static and dynamic content. Static content does not change. However, dynamic content is typically content that is created by an application and therefore changes often, such as Active Server Pages (ASP) or ASP.NET content. Since dynamic content should change often, IIS 7.x does not cache it. If you enable this feature, you should test it very well since it might hamper the performance of your site. Dynamic compression will occur on every request. Imagine this situation on a site with a large number of visitors!!

Static files are physical files like .css and .txt that exist on disk. It makes sense to compress this type of files before sending them because they contain textual data and their size will reduce significantly. IIS caches compressed static content in a special directory (e.g. `%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files`), which increases compression performance by eliminating the need to recompress content that has already been compressed. After IIS has compressed a file, subsequent requests are given the compressed copy of the file from the cache directory.

Not all file types are suitable for compression, this includes .png and .jpg files, which are already compressed by nature. Trying to compress these types of files is a waste of time and CPU. Do not worry, IIS knows what types of files to compress and what not to. If you want to take a look or configure file types that are to be compressed, open `ApplicationHost.config` file and navigate to the `<httpcompression>` section:

```xml


<httpCompression directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
            <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll" />
            <dynamicTypes>
                <add mimeType="text/*" enabled="true" />
                <add mimeType="message/*" enabled="true" />
                <add mimeType="application/x-javascript" enabled="true" />
                <add mimeType="*/*" enabled="false" />
            </dynamicTypes>
            <staticTypes>
                <add mimeType="text/*" enabled="true" />
                <add mimeType="message/*" enabled="true" />
                <add mimeType="application/x-javascript" enabled="true" />
                <add mimeType="application/javascript" enabled="true" />
                <add mimeType="application/atom+xml" enabled="true" />
                <add mimeType="application/xaml+xml" enabled="true" />
                <add mimeType="*/*" enabled="false" />
            </staticTypes>
        </httpCompression>
```

You can even add your custom MIME types if needed.

When setting compression settings at server level, you have more options. You can modify parameters like directory path for compressed static files, minimum file size to compress and maximum size of that directory.

![](/images/posts/archived/http-compression-1.png)

**NOTE**: If you navigate to the compressed directory and try to open a compressed file with 3rd party tools like WinRar, you will get an error message. Although these tools (e.g. WinRar) do support the compression format, but they expect the files to contain metadata like header, which your files do not. Of course, it is not impossible to read these files, you can easily write a C# program to read them.

### HTTP Compression in Action

The easiest way to test HTTP Compression in action is by using FireBug. If you haven't installed this nice plug-in yet, then I recommend that you do it now. Launch Firebug and type the following command

```javascript
 $.get("/jquery-1.4.2.min.js")
 ```

To determine if the response was compressed, Click on the "headers" tab, and look at the Content-Encoding value under "response". The presence of this value indicates response was compressed and the value indicates the compression method, which is gzip in this case. Note that the original file size is 70.4 KB, and its size after compression is 24KB. You saved 66% which is good enough.

Even if .js and .css are compressed across the wire, this does not mean you should leave out comments and whitespaces inside them. Always strip all whitespaces and comments from your files on production servers for security reasons, too. Comments have proven to give malicious users tracks to exploit your site.