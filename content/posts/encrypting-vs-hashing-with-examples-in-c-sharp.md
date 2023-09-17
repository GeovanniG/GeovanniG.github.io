---
title: "Encrypting vs Hashing With Examples In C#"
date: 2023-09-17T00:00:00-07:00
draft: false
---

Encrypting and hashing refer to two different concepts, yet occasionally, they are mistakenly used interchangeably. I, for one, have made the mistake of confusing encryption with hashing and vice versa. To gain a better understanding of the two concepts, we describe their similarities, differences, and use cases.

## What Is Encryption?

Encryption is a process of masking data. Its main objective is to protect data confidentiality. The data is masked using a key and only those with access to the key are able to decrypt the data. Therefore, to ensure data confidentiality, the keys must be secured and only shared when absolutely necessary.

Encryption supports 2 ways of encrypting / decrypting data: symmetrically and asymmetrically. With symmetric encryption, the same key is used to encrypt and decrypt the data. Since the key is used for encryption and decryption, it must be protected at all costs. For security reasons, it's best to use symmetric encryption when encrypting data within internal systems. With asymmetric encryption, there are two keys involved: a public and a private key. The encryption is performed using the public key, which is allowed to be distributed to clients. While the decryption is performed using the private key, which must not be shared. When communicating with less-trusted sources and encryption is required, it is best to use asymmetric encryption.

## What is Hashing?

Hashing also involves masking data. However, its main objective is to ensure data integrity and verify the authenticity of data. Hashing is not concerned with decrypting the masked data, and in fact, it is impossible to decrypt a hashed value.

To ensure data integrity and authenticity of the data, hashing must be deterministic, meaning the same input must always yield the same output. Furthermore, collisions, where two different inputs yield the same hash value, must be minimized or nonexistent. Note, with the use of modern hashing functions such as SHA-256, collisions are very unlikely. Therefore, if these conditions are satisfied and the two hashed values are equal, we can confidently say the inputs are also equal. By comparing hash values in this way, we can determine the authenticity of the input data.

## When To Use Encryption or Hashing?

Encryption is used when we would like to securely send data to an intended audience. Assuming the intended audience has the decryption key, they will be able to decrypt and read the data. If the data is obtained by an unexpected party, as long as they do not have the key, they will be unable to read the data. A great example of encryption is communication over HTTPS. The server shares the public key with the client. The data is encrypted by the client using the public key and decrypted by the server using the private key.

Hashing, on the other hand, is meant for comparison. For example, when a user logs into our application, it is better to hash the password, than it is to encrypt it. When authenticating a user, our main objective is to verify the user's credentials. Instead of storing the password in our database, we can store the hash of the password, and compare the hash of the entered password to the hash in our database. If they match, the user is successfully authenticated. Otherwise, they are not. If we mistakenly make the decision to store encrypted passwords, and our database is compromised, a malicious user has the potential to decrypt all of our user's passwords! With hashing, since it is impossible to decrypt, this will never happen.

## Code Implementation

We will only cover symmetric encryption. To encrypt and decrypt data in C# we will need:
* the key used to encrypt / decrypt the data,
* the algorithm used for the encryption / decryption, and
* the `CryptoStream` to write and read the encrypted / decrypted data. 

```c#
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class AESEncryptionExample
{
    public static string Encrypt(string plainText, string key, string iv)
    {
        try
        {
            using (Aes aesAlg = Aes.Create())
            {
                aesAlg.Key = Encoding.UTF8.GetBytes(key);
                aesAlg.IV = Encoding.UTF8.GetBytes(iv);

                ICryptoTransform encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);

                using (MemoryStream msEncrypt = new MemoryStream())
                {
                    using (CryptoStream csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                    {
                        using (StreamWriter swEncrypt = new StreamWriter(csEncrypt))
                        {
                            swEncrypt.Write(plainText);
                        }
                    }

                    return Convert.ToBase64String(msEncrypt.ToArray());
                }
            }
        }
        catch (Exception ex)
        {
            // Handle exceptions, log, and possibly notify administrators.
            Console.WriteLine("Encryption Error: " + ex.Message);
            return null;
        }
    }

    public static string Decrypt(string cipherText, string key, string iv)
    {
        try
        {
            using (Aes aesAlg = Aes.Create())
            {
                aesAlg.Key = Encoding.UTF8.GetBytes(key);
                aesAlg.IV = Encoding.UTF8.GetBytes(iv);

                ICryptoTransform decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);

                using (MemoryStream msDecrypt = new MemoryStream(Convert.FromBase64String(cipherText)))
                {
                    using (CryptoStream csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
                    {
                        using (StreamReader srDecrypt = new StreamReader(csDecrypt))
                        {
                            return srDecrypt.ReadToEnd();
                        }
                    }
                }
            }
        }
        catch (Exception ex)
        {
            // Handle exceptions, log, and possibly notify administrators.
            Console.WriteLine("Decryption Error: " + ex.Message);

            return null;
        }
    }

    public static void Main(string[] args)
    {
        try
        {
            string key = "0123456789ABCDEF"; // 128-bit key
            string iv = "1234567890ABCDEF";  // 128-bit IV

            string plainText = "This is a secret message.";
            string encryptedText = Encrypt(plainText, key, iv);

            if (encryptedText != null)
            {
                Console.WriteLine("Encrypted: " + encryptedText);

                string decryptedText = Decrypt(encryptedText, key, iv);
                if (decryptedText != null)
                {
                    Console.WriteLine("Decrypted: " + decryptedText);
                }
                else
                {
                    Console.WriteLine("Decryption failed.");
                }
            }
            else
            {
                Console.WriteLine("Encryption failed.");
            }
        }
        catch (Exception ex)
        {
            // Handle any unhandled exceptions, log, and possibly notify administrators.
            Console.WriteLine("An unexpected error occurred: " + ex.Message);
        }
    }
}
```

In a more real-world example, we would store our password in a secure location such as Azure Key Vault. For those interested in asymmetric encryption please see [Asymmetric Encryption](https://learn.microsoft.com/en-us/dotnet/standard/security/encrypting-data#asymmetric-encryption) and [Asymmetric Decryption](https://learn.microsoft.com/en-us/dotnet/standard/security/decrypting-data#asymmetric-decryption).

Hashing is also straight-forward to accomplish. It mainly involves converting our input into bytes and calling the hash function. For example, we use the `SHA256` hash function.

```c#
using System;
using System.Security.Cryptography;
using System.Text;

public class HashingExample
{
    public static string CalculateSHA256Hash(string input)
    {
        using (SHA256 sha256 = SHA256.Create())
        {
            byte[] bytes = Encoding.UTF8.GetBytes(input);
            byte[] hashBytes = sha256.ComputeHash(bytes);
            StringBuilder builder = new StringBuilder();

            foreach (byte b in hashBytes)
            {
                builder.Append(b.ToString("x2"));
            }

            return builder.ToString();
        }
    }

    public static void Main(string[] args)
    {
        string data = "Hello, world!";
        string sha256Hash = CalculateSHA256Hash(data);
        Console.WriteLine("SHA-256 Hash: " + sha256Hash);
    }
}
```

Thanks for reading!