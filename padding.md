# Padding

## Purpose

The length of a ciphertext from a [stream cipher](authenticated-encryption/chacha20-poly1305.md) is equal to the length of the plaintext. In most cases, this is not considered an issue. However, hiding the length of a message can be desirable, and [ISO/IEC 7816-4](https://en.wikipedia.org/wiki/Padding_\(cryptography\)#ISO/IEC_7816-4) padding can be used to do this.

The amount of padding, determined by the block size, can either be deterministic or [randomised](random-data.md#getint32). Both have their [strengths and weaknesses](https://en.wikipedia.org/wiki/Padding_\(cryptography\)#Traffic_analysis_and_protection_via_padding).

{% hint style="warning" %}
Padding to a block size much smaller than the message length leaves the approximate unpadded length largely unprotected. [PADMÉ](https://github.com/samuel-lucas6/PADME.NET) can be used to limit leakage.
{% endhint %}

{% hint style="success" %}
Padding should be applied to the plaintext before encryption and removed from the plaintext after decryption. The amount of padding does not need to be stored.
{% endhint %}

## Usage

### Fill

Fills a span with padding. This can then be [manually concatenated](advanced/concat.md) with some data.

```csharp
Iso78164Padding.Fill(Span<byte> buffer)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

### GetPaddedBufferSize

Returns the required buffer size for `Pad()` based on the unpadded data length and a block size (e.g., 16 bytes).

```csharp
Iso78164Padding.GetPaddedBufferSize(ReadOnlySpan<byte> data, int blockSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

`data.Length + paddingSize` has resulted in an overflow.

### Pad

Fills a span with the data padded up to the specified block size (e.g., a multiple of 16 bytes).

```csharp
Iso78164Padding.Pad(Span<byte> buffer, ReadOnlySpan<byte> data, int blockSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `GetPaddedBufferSize(data, blockSize)`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[CryptographicException](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error padding data.

### GetUnpaddedBufferSize

Returns the number of bytes to slice from the end of the padded data.

```csharp
Iso78164Padding.GetUnpaddedBufferSize(ReadOnlySpan<byte> paddedData, int blockSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`paddedData` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid padding.

## Notes

{% hint style="warning" %}
It is very difficult to hide that cryptography is being used. For example, even if padding is done appropriately and there are no plaintext headers, [X25519](key-exchange.md) public keys are [distinguishable from random](https://elligator.org/).
{% endhint %}

{% hint style="warning" %}
Using padding to hide the length of a password is **NOT** recommended. Instead, the password can be prehashed using [BLAKE2b](hashing.md) or [Argon2id](password-hashing.md#notes) on the client before being sent to the server for password hashing.
{% endhint %}
