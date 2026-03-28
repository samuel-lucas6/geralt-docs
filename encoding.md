# Encoding

## Purpose

It can be useful to convert bytes to strings. For example, to represent [hashes](hashing.md) or to create shareable [X25519](key-exchange.md) and [Ed25519](digital-signatures.md) public keys. Hex and Base64 encoding can be used to do this.

## Usage

### ToHex

Returns the hexadecimal string that represents the data.

```csharp
Encodings.ToHex(ReadOnlySpan<byte> data)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

The data could not be converted to hex. This should never be thrown.

### FromHex

Returns a byte array from a hexadecimal string. Common separator characters are ignored by default. `ignoreChars` can be `null` to disallow any non-hexadecimal characters.

```csharp
Encodings.FromHex(string hex, string ignoreChars = ":- ")
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`hex` is null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hex` has a length of 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Unable to parse the hex string.

### ToBase64

Returns the Base64 string representing the data. Choose one variant (e.g. Base64URL for file names/URLs) and only ever use that variant.

```csharp
Encodings.ToBase64(ReadOnlySpan<byte> data, Base64Variant variant = Base64Variant.Original)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

The data could not be converted to Base64. This should never be thrown.

### FromBase64

Returns a byte array from a Base64 string. The variant must match the one used for encoding. `ignoreChars` can be `null` to disallow any non-Base64 characters.

```csharp
Encodings.FromBase64(string base64, Base64Variant variant = Base64Variant.Original, string ignoreChars = " ")
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`base64` is null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`base64` has a length of 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Unable to parse the Base64 string.

## Constants

```csharp
public const string HexCharacterSet = "0123456789ABCDEFabcdef";
public const string Base64CharacterSet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
public const string Base64UrlCharacterSet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_=";
public const string Base64FullCharacterSet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/-_=";
public const string HexIgnoreChars = ":- _./,%;";
public const string Base64IgnoreChars = " \r\n";

public enum Base64Variant
{
    Original = 1,
    OriginalNoPadding = 3,
    Url = 5,
    UrlNoPadding = 7
}
```

## Notes

{% hint style="danger" %}
Do **NOT** support multiple variants of Base64 in your application (e.g. Base64 with and without padding). This can lead to [vulnerabilities](https://cendyne.dev/posts/2022-02-18-user-provided-primary-keys.html).
{% endhint %}

{% hint style="success" %}
Unlike in the .NET library, these functions run in constant time to avoid potential [side-channel attacks](https://en.wikipedia.org/wiki/Side-channel_attack). Furthermore, the Base64 implementation should be more resistant to [malleability attacks](https://dl.acm.org/doi/10.1145/3488932.3527284).
{% endhint %}

{% hint style="info" %}
Base64 has a better compression rate than hex. However, they tend to be used for different purposes. For instance, hex is often used for [test vectors](https://datatracker.ietf.org/doc/html/rfc8439#appendix-A) and [encoding hashes](https://crypto.stackexchange.com/questions/34995/why-do-we-use-hex-output-for-hash-functions), whereas Base64 is more commonly used for [encoding keys](https://www.wireguard.com/quickstart/#key-generation).
{% endhint %}
