---
layout: post
title: Restoring Classic Calculator in Windows 10
redirect_from:
- "/post/restoring-classic-calculator-in-windows-10.aspx.html"
tags:
- calc
- calculator
- windows 10
---
## Restoring Classic Calculator in Windows 10

I recently installed a new version of Windows 10 and noticed that Windows Calculator interface has changed dramatically. 
![Windows 10 Calculator](/images/posts/archived/restoring-classic-calculator-in-windows-10-1.png "Windows 10 Calculator")

I personally didn't like. It took me a while to figure out how to perform inverse trignometric functions. Finally, I realized their buttons only show up when the window size is wide enough! 

![Old Calculator](/images/posts/archived/restoring-classic-calculator-in-windows-10-2.png "Old Calculator")

So I decided to restore the classic Calcualtor app. If would like the classic Calculator interface back, you will need copy the classic Windows Calculator files from an earlier version of Windows 10, or from Windows 8 or 7 (this works too). I attached the needed files to this post. If you don't trust these, you have no choice but to get them manually.

* Two files are needed from an earlier version of Windows: `%SystemRoot%\system32\calc.exe` and `%SystemRoot%\system32\en-US\calc.exe.mui`.
Note that the path might be slightly different in your case.
* Download [calc.zip](/attachments/posts/archived/restoring-classic-calculator-in-windows-10-1.zip) and extract the files to a folder such as `C:\calc\`.
* If you copied the two mentioned files manually, then place them in the same folder. 
Make sure you name the files `calc.exe` and `calc.exe.mui`.
* Run command prompt as **Administrator** by right clicking on Command prompt and then choosing 'Run as administrator'.
* Run the file `install.bat` and press `y` when prompted.
You might need to tweak the paths in install.bat if your Windows version is not en-US. 
*You should see the following output if every thing went well. 
```
C:\Windows\system32>C:\calc\install.bat
Taking ownership of calc.exe...
SUCCESS: The file (or folder): "C:\Windows\system32\calc.exe" now owned by the administrators group.
Are you sure (Y/N)?y
processed file: C:\Windows\system32\calc.exe
Taking ownership of calc.exe.mui...
SUCCESS: The file (or folder): "C:\Windows\system32\en-US\calc.exe.mui" now owned by the administrators group.
Are you sure (Y/N)?y
Copying calc.exe...
        1 file(s) copied.
Copying calc.exe.mui...
        1 file(s) copied.
```
* Enjoy!
