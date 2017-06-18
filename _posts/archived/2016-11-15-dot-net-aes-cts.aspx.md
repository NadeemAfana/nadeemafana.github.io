---
layout: post
title: .NET AES CTS Implementation
redirect_from:
- "post/dot-net-aes-cts.aspx.html"
tags:
- methods
- encryption
- cryptography
- .net
- AES
- CBC
- CTS
---
## .NET AES CTS Implementation
I was looking for an AES CTS .NET implementation but could not find any. The .NET Framework has an enumeration value for CTS; however, the framework throws an exception if you try to use it as it is not actually implemented. I then decided to write one and share it: 

Install using NuGet

`Install-Package AesCts`

```csharp
// using Afana.Cryptography.Aes
void Main()
{
    byte[] plainText = { 0x0c, 0x00, 0x04, 0x00, 0x0c, 0x00, 0x18, 
                        0x00, 0x00, 0x00, 0x00, 0x00, 0x22, 0x00, 
                        0x04, 0xa};
    byte[] iv = { 0x4a, 0xcd, 0xca, 0x4f, 0xa8, 0xb3, 0x51, 0x8b, 
                  0x8b, 0xd0, 0x39, 0xc3, 0x0c, 0x61, 0xad, 0xbf };
 
    byte[] key = { 0xed, 0xb0, 0xb0, 0x3b, 0x59, 0xf2, 0xf5, 0xe7, 
                   0x4c, 0x04, 0xe5, 0xb8, 0xcd, 0x56, 0x40, 0x5c, 
                   0xed, 0xb0, 0xb0, 0x3b, 0x59, 0xf2, 0xf5, 0xe7, 
                   0x4c, 0x04, 0xe5, 0xb8, 0xcd, 0x56, 0x40, 0x5c };
     
    byte[] cipherText =  AesCts.Encrypt(plainText, key, iv);
    Console.WriteLine(cipherText.ToHexString()); 
        // For .ToHexString(), Install-Package ExtensionMethods
     
    // Outputs fc9112f1edc30d68544b2951612e66c3
}
```

