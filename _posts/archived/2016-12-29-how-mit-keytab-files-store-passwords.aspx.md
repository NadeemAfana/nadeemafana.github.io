---
layout: post
title: How MIT Keytab Files Store Passwords
redirect_from:
- "/post/how-mit-keytab-files-store-passwords.aspx.html"
tags:
- kerberos
- security
- nfold
- aes
- cts
- sha
- hmac
- rfc3961
- rfc3962
---
## How Kerberos Keys Are Generated
A Keytab file is a way to store your passwords so that you can authenticate to Kerberos services without typing your password again. It is similar to logon tokens in a way. I initially thought storing passwords in a keytab file was straightforward, but once I started digging into the internals, I realized I was wrong, well, half wrong because in the case of `RC4-HMAC`, it is so simple.

In general, the password is not stored in raw format. A derived key is generated based on the password, and the key is stored instead.

here are multiple encryption types that Keytabs support, however, I will discuss only the following formats, which are the most common anyways:

1. RC4-HMAC
2. AES-128-CTS-HMAC-SHA1-96
3. AES-256-CTS-HMAC-SHA1-96

## RC4-HMAC
This is the simplest case:
```
Key = Md4 (Unicode(password))
```
The key is simply the MD4 hash of the password.

The following C# code will compute the key given a password.
```csharp
// Packages required
// Install-Package ExtensionMethods
// Install-Package Afana.Cryptography
void Main()
{
     
    byte[] key = GetRc4HmacKey("password");
    Console.WriteLine(key.ToHexString());
    // outputs 8846f7eaee8fb117ad06bdd830b7586c
     
    // Note: For .ToHexString() and other extension methods, Install-Package ExtensionMethods.
     
}

byte[] GetRc4HmacKey(string password)
{
    using (HashAlgorithm hash = new Md4()) {
        return hash.ComputeHash(password.ToBytes(Encoding.Unicode));
    }
}    
```
## AES
When it comes to AES, things are more complicated than a simple hash. Although key generation is documented in `RFC3962`, the process is hard to follow through and is rather complicated.

## AES-128-CTS-HMAC-SHA1-96
The encryption algorithm used by `AES-128-CTS-HMAC-SHA1-96` is `AES CBC CTS` with 128-bits key size. CTS, **Cipher Text Stealing**, generates a cipher text which has exactly the same size as the input plain text. CTS won't matter in our case since the input data size is exactly 16 octets, one block, so we could simply use AES CBC instead. 

The slightly simplified formula for generating the AES 128-bit key is:
```
tkey = random2key(PBKDF2(passphrase, salt, iter_count, keylength))
key = E(tkey, NFold("kerberos"), initial-cipher-state))
E = Encrypt using AES-128-CBC-CTS
```
`random2key` is an identity function, which means it returns the input as is, so this function can be safely ignored.

`PBKDF2` is a standard function (RFC2898) that produces a derived key given a password. Its purpose is to make password cracking much more difficult. 

`passphrase` is the password itself. `salt` is the realm + user name. `iter_count` is 4,096. `keylength` is 16 bytes (128 bits). 

AES CBC works in terms of blocks, which means if the data to be encrypted is less than one block in size, it must be padded. Because the string "kerberos" is less than one block (16 bytes), it must be expanded and NFold is what expands it to 16 bytes. NFold is a function that takes in a variable-length input and returns a fixed-length output. The .NET Framework does not provide an implementation of n-folding, so I had to write my own.

Here is a fully working C# example that computes the key:
```csharp
// Packages required
// Install-Package ExtensionMethods
// Install-Package Afana.Cryptography
 
void Main()
{
     
    byte[] key = GetAes128Sha1Key("password", "nadeem", "MY.REALM.COM");
    Console.WriteLine(key.ToHexString());
    // outputs 907e1f9fe6fa7359543f296d6552444c
     
    // Note: For .ToHexString(), Install-Package ExtensionMethods.
     
}
 
byte[] GetAes128Sha1Key(string password, string userName, string realm)
{
    // AES-128-CTS-HMAC-SHA1-96
    const int keySize = 16;
    byte[] pbkdf2 = ComputePbkdf2(password.ToBytes(), (realm+userName).ToBytes(), keySize); 
    byte[] nfolded = NFold.Compute("kerberos".ToBytes(), keySize);
    return AesCts.Encrypt(nfolded, pbkdf2, new byte[keySize]);
}
 
// Pbdkf2 based on HMACSHA1
public static byte[] ComputePbkdf2(byte[] password, byte[] salt, int size)
{
    using (Rfc2898DeriveBytes pbkdf2 = new Rfc2898DeriveBytes(password, salt, 4096)) {
        // Truncate hash to "size" bytes.
        return pbkdf2.GetBytes(size);
    }
}
```
## AES-256-CTS-HMAC-SHA1-96

The encryption algorithm used by AES-256-CTS-HMAC-SHA1-96 is AES CBC CTS with 256-bits key size.

Unlike AES 128, there a few extra steps for key generation:

```
tkey = random2key(PBKDF2(passphrase, salt, iter_count, keylength))
k1 = E(tkey, NFold("kerberos"), initial-cipher-state))
k2 = E(tKey, k1, initial-cipher-state)
Key = k1 || k2
E = Encrypt using AES-256-CBC-CTS
```

```csharp
// Packages required
// Install-Package ExtensionMethods
// Install-Package Afana.Cryptography
 
void Main()
{
    byte[] key = GetAes256Sha1Key("password", "nadeem", "MY.REALM.COM");
    Console.WriteLine(key.ToHexString());
    // outputs 08a29fccbc66d60f95ec6122545844adacff001393d79640d3ba3da17562ebe6
    // Note: For .ToHexString(), Install-Package ExtensionMethods.
}
 
byte[] GetAes256Sha1Key(string password, string userName, string realm)
{
    // AES-256-CTS-HMAC-SHA1-96
    byte[] pbkdf2 = ComputePbkdf2(password.ToBytes(), (realm + userName).ToBytes(), 32);    
    byte[] nfolded = NFold.Compute("kerberos".ToBytes(), 16);
    byte[] iv = new byte[16];
    byte[] k1 = AesCts.Encrypt(nfolded, pbkdf2, iv);
    byte[] k2 = AesCts.Encrypt(k1, pbkdf2, iv);
    // k1 || k2 
    return k1.Concat(k2).ToArray();
}
 
// Pbdkf2 based on HMACSHA1
public static byte[] ComputePbkdf2(byte[] password, byte[] salt, int outputSize)
{
    using (Rfc2898DeriveBytes pbkdf2 = new Rfc2898DeriveBytes(password, salt, 4096)) {
        // Truncate hash to "outputSize" bytes.
        return pbkdf2.GetBytes(outputSize);
    }
}  
```