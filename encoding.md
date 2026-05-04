# Encoding

## Purpose

It can be useful to convert bytes to strings. For example, to represent [hashes](hashing.md) or to create shareable [X25519](key-exchange.md) and [Ed25519](digital-signatures.md) public keys. Hex and Base64 encoding can be used to do this.

## Usage

### GetToHexBufferSize

Returns the output buffer size required for `ToHex()`.

```csharp
Encodings.GetToHexBufferSize(ReadOnlySpan<byte> data)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

`data.Length * 2` has resulted in an overflow.

### ToHex

Fills a span with the hexadecimal string that represents the provided data.

```csharp
Encodings.ToHex(Span<char> hex, ReadOnlySpan<byte> data)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hex` has a length not equal to `GetToHexBufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

`data.Length * 2` has resulted in an overflow.

[CryptographicException](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error converting bytes to hex.

### GetFromHexBufferSize

Returns the output buffer size required for `FromHex()`.

```csharp
Encodings.GetFromHexBufferSize(ReadOnlySpan<char> hex, ReadOnlySpan<char> ignoreChars = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hex` has a length of 0 or is not a multiple of 2 (after discounting ignored chars).

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`ignoreChars` cannot contain hex characters or represent the only characters in `hex`.

### FromHex

Fills a span with the data from decoding a hexadecimal string. Separator characters to ignore when parsing can optionally be provided.

```csharp
Encodings.FromHex(Span<byte> data, ReadOnlySpan<char> hex, ReadOnlySpan<char> ignoreChars = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length not equal to `GetFromHexBufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hex` has a length of 0 or is not a multiple of 2 (after discounting ignored chars).

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`ignoreChars` cannot contain hex characters or represent the only characters in `hex`.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid hex string.

### GetToBase64BufferSize

Returns the output buffer size required for `ToBase64()`.

```csharp
Encodings.GetToBase64BufferSize(ReadOnlySpan<byte> data, Base64Variant variant = Base64Variant.Original)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

The encoded length is too large for an array.

### ToBase64

Fills a span with the Base64 string that represents the provided data. Choose one variant (e.g., Base64URL for file names/URLs) and only ever use that variant.

```csharp
Encodings.ToBase64(Span<char> base64, ReadOnlySpan<byte> data, Base64Variant variant = Base64Variant.Original)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`base64` has a length not equal to `GetToBase64BufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length of 0.

[CryptographicException](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error converting bytes to Base64.

### GetFromBase64BufferSize

Returns the output buffer size required for `FromBase64()`.

```csharp
Encodings.GetFromBase64BufferSize(ReadOnlySpan<char> base64, Base64Variant variant = Base64Variant.Original, ReadOnlySpan<char> ignoreChars = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`base64` has a length of 0 or is not a valid length based on `variant`.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`ignoreChars` cannot contain Base64 characters or represent the only characters in `base64`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

The buffer size computation has resulted in an overflow.

### FromBase64

Fills a span with the data from decoding a Base64 string. The variant must match the one used for encoding. Separator characters to ignore when parsing can optionally be provided.

```csharp
Encodings.FromBase64(Span<byte> data, ReadOnlySpan<char> base64, Base64Variant variant = Base64Variant.Original, ReadOnlySpan<char> ignoreChars = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`data` has a length not equal to `GetFromBase64BufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`base64` has a length of 0 or is not a valid length based on `variant`.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`ignoreChars` cannot contain Base64 characters or represent the only characters in `base64`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

The buffer size computation has resulted in an overflow.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid Base64 string.

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
Do **NOT** support multiple variants of Base64 in your application (e.g., Base64 with and without padding). This can lead to [vulnerabilities](https://cendyne.dev/posts/2022-02-18-user-provided-primary-keys.html).
{% endhint %}

{% hint style="success" %}
Unlike in the .NET library, these functions run in constant time to avoid potential [side-channel attacks](https://en.wikipedia.org/wiki/Side-channel_attack). Furthermore, the Base64 implementation should be more resistant to [malleability attacks](https://dl.acm.org/doi/10.1145/3488932.3527284).
{% endhint %}

{% hint style="info" %}
Base64 has a better compression rate than hex. However, they tend to be used for different purposes. For instance, hex is often used for [test vectors](https://datatracker.ietf.org/doc/html/rfc8439#appendix-A) and [encoding hashes](https://crypto.stackexchange.com/questions/34995/why-do-we-use-hex-output-for-hash-functions), whereas Base64 is more commonly used for [encoding keys](https://www.wireguard.com/quickstart/#key-generation).
{% endhint %}
