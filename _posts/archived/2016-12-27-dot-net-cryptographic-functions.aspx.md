---
layout: post
title: .NET Cryptographic Functions
redirect_from:
- "post/dot-net-cryptographic-functions.aspx.html"
tags:
- methods
- encryption
- cryptography
- .net
- AES
- CBC
- CTS
- Lcm
- Gcd
- Ror
- Rol
- Rotation
- NFold
---
## Cryptographic functions
`Afana.Cryptography` Nuget package is meant to be a complementary set to the .NET Framework. It is not a replacement. It is still work in progress, and I will add more functions if I receive enough requests from people. For example, the .NET Framework has no implementation of `AES CTS` or `N-Fold`. 

To add the library to your project, open Package Manager Console, select your project and type
`Install-Package Afana.Cryptography`

The following functions are available so far
## AES CBC CTS
```csharp
// using Afana.Cryptography.Aes
void Main()
{
    byte[] plainText = { 0x0c, 0x00, 0x04, 0x00, 0x0c, 0x00, 0x18, 
                         0x00, 0x00, 0x00, 0x00, 0x00, 0x22, 0x00, 
                         0x04, 0xa
                       };
    byte[] iv = { 0x4a, 0xcd, 0xca, 0x4f, 0xa8, 0xb3, 0x51, 0x8b, 
                  0x8b, 0xd0, 0x39, 0xc3, 0x0c, 0x61, 0xad, 0xbf 
                };
    byte[] key = { 0xed, 0xb0, 0xb0, 0x3b, 0x59, 0xf2, 0xf5, 0xe7, 
                   0x4c, 0x04, 0xe5, 0xb8, 0xcd, 0x56, 0x40, 0x5c, 
                   0xed, 0xb0, 0xb0, 0x3b, 0x59, 0xf2, 0xf5, 0xe7, 
                   0x4c, 0x04, 0xe5, 0xb8, 0xcd, 0x56, 0x40, 0x5c 
                 };
     
    byte[] cipherText =  AesCts.Encrypt(plainText, key, iv);
     
    Console.WriteLine(cipherText.ToHexString()); 
    // For .ToHexString(), Install-Package ExtensionMethods
     
    // Outputs fc9112f1edc30d68544b2951612e66c3
}
```

## Rotate Left
You can rotate left an array of bytes, not only integers.
```csharp
// using Afana.Cryptography
void Main() 
{
    byte[] value = new byte[] { 0xa, 0xb, 0xc, 0xd };
    byte[] result = CryptoHelper.RoL(value, 5);
     
    Console.WriteLine(result.ToHexString()); 
    // Outputs 416181a1 
     
    // Note: For .ToHexString(), Install-Package ExtensionMethods
}   
```

## Rotate Right
Rotates right an array of bytes, not only integers.
```csharp
// using Afana.Cryptography
void Main() 
{
    byte[] value = new byte[] { 0xa, 0xb, 0xc, 0xd };
    byte[] result = CryptoHelper.RoR(value, 5);
     
    Console.WriteLine(result.ToHexString()); 
    // Outputs 68505860 
     
    // Note: For .ToHexString(), Install-Package ExtensionMethods
}
```

## Greatest Common Divisor

Calculates the Greatest Common Divisor for two integers.
```csharp
// using Afana.Cryptography
void Main() 
{
      
    int result = CryptoHelper.Gcd(-15, 21); 
    Console.WriteLine(result);
    // Outputs 3
 
    result = CryptoHelper.Gcd(2, 4);
    Console.WriteLine(result);
    // Outputs 2        
}
```

## Least Common Multiple

Calculates the Least Common Multiple for two integers.
```csharp
// using Afana.Cryptography
void Main() 
{
      
    int result = CryptoHelper.Lcm(-15, 21); 
    Console.WriteLine(result);
    // Outputs 105  
 
}
```

## N-Folding

N-Folds an array of bytes. See RFC3961 for more information.
```csharp
// using Afana.Cryptography
void Main() 
{
    byte[] plainText = "kerberos".ToBytes();    
    byte[] nfolded = NFold.Compute(plainText, 32);
 
    Console.WriteLine(nfolded.ToHexString());
    // Outputs 6b65726265726f737b9b5b2b93132b935c9bdcdad95c9899c4cae4dee6d6cae4 
 
    // Note: For .ToHexString(), Install-Package ExtensionMethods
}
```