# ISO/IEC 7816-4

## Purpose

The length of a ciphertext from a [stream cipher](../authenticated-encryption/chacha20-poly1305.md) is equal to the length of the plaintext. Similarly, [block cipher padding](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Padding) only adds a small overhead. This metadata can leak information about what was encrypted and thus is something protocols may want to hide.

[ISO/IEC 7816-4](https://en.wikipedia.org/wiki/Padding_\(cryptography\)#ISO/IEC_7816-4) padding adds a mandatory hex `80` byte, followed by `00` bytes (if needed) up to a multiple of the block size. This makes it reversible compared to all-zero padding. However, this algorithm does not help with figuring out how much padding to add/what block size to use.

{% hint style="warning" %}
Padding to a block size much smaller than the message length leaves the approximate unpadded length unprotected. [Padmé](padme.md) can be used to limit leakage. However, it is important to be aware of its limitations and the [strengths/weaknesses](https://en.wikipedia.org/wiki/Padding_\(cryptography\)#Traffic_analysis_and_protection_via_padding) of deterministic versus randomised padding.
{% endhint %}

{% hint style="success" %}
Padding should be applied to the plaintext before encryption and removed from the plaintext after decryption. The amount of padding does not need to be stored when using this algorithm.
{% endhint %}

## Usage

### Fill

Fills a span with padding. This can then be [manually concatenated](../advanced/concat.md) with some data.

```csharp
Iso78164Padding.Fill(Span<byte> buffer);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is equal to 0.

### GetPaddedBufferSize

Returns the required buffer size for `Pad()` based on the unpadded data length and a block size (e.g., 16 bytes).

```csharp
int bufferSize = Iso78164Padding.GetPaddedBufferSize(ReadOnlySpan<byte> data, int blockSize);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

`data.Length + paddingSize` has resulted in an overflow.

### Pad

Fills a span with the data padded up to the specified block size (e.g., a multiple of 16 bytes).

```csharp
Iso78164Padding.Pad(Span<byte> buffer, ReadOnlySpan<byte> data, int blockSize);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is not equal to `GetPaddedBufferSize(data, blockSize)`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

`GetPaddedBufferSize()` validation has resulted in an overflow.

[CryptographicException](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error padding data.

### GetUnpaddedBufferSize

Returns the number of bytes to slice from the end of the padded data.

```csharp
int unpaddedSize = Iso78164Padding.GetUnpaddedBufferSize(ReadOnlySpan<byte> paddedData, int blockSize);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`paddedData.Length` is equal to 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`blockSize` is less than or equal to 0.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid padding.

## Notes

{% hint style="warning" %}
Using padding to hide the length of a password is **NOT** recommended. Instead, the password can be prehashed using [BLAKE2b](../hashing.md) or [Argon2id](../password-hashing.md#notes) on the client before being sent to the server for password hashing.
{% endhint %}

{% hint style="success" %}
The pad and unpad functions try to minimise side channels.
{% endhint %}
