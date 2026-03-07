# AEGIS-256

## Purpose

[AEGIS-256](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-aegis-aead) is an AES-based authenticated encryption with associated data (AEAD) scheme that was a [CAESAR competition](https://competitions.cr.yp.to/caesar-submissions.html) finalist. It encrypts a plaintext message using a 256-bit key and nonce (**number used only once**) whilst calculating a 256-bit tag over the plaintext and associated data.

The associated data is useful for authenticating file headers, version numbers, timestamps, counters, and so on. It can be used to prevent [confused deputy attacks](https://cloud.google.com/kms/docs/additional-authenticated-data#confused_deputy_attack_example) and [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). It is not encrypted nor part of the ciphertext. It must be reproduceable or stored somewhere for decryption to be possible.

Decryption involves verifying the tag for the given inputs, which detects tampering and incorrect parameters. If verification fails, an error is returned. Otherwise, the plaintext is returned.

{% hint style="danger" %}
For encryption, the nonce **MUST NOT** be repeated or reused with the same key. You **MUST** [increment](../constant-time.md#increment) or [randomly generate](../random-data.md#fill) the nonce for each plaintext message encrypted using the same key.
{% endhint %}

{% hint style="success" %}
Unlike with [ChaCha20-Poly1305](chacha20-poly1305.md), it is safe to [randomly generate](../random-data.md#fill) nonces with the same key. Nonces can be public and are typically manually prepended to the ciphertext.

It is not necessary to use the full 256-bit nonce. For example, a 160- or 192-bit random nonce **MAY** be used, with zero padding up to 256 bits. However, a 256-bit nonce ensures there is no practical limit on the number of messages that can be encrypted using the same key.
{% endhint %}

## Usage

### Encrypt

Fills a span with ciphertext and an appended tag computed from a plaintext message, nonce, key, and optional associated data.

```csharp
AEGIS256.Encrypt(Span<byte> ciphertext, ReadOnlySpan<byte> plaintext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ReadOnlySpan<byte> associatedData = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `plaintext.Length + TagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Encryption failed.

### Decrypt

Verifies that the tag appended to the ciphertext is correct for the given inputs. If verification fails, an exception is thrown. Otherwise, it fills a span with the decrypted ciphertext.

```csharp
AEGIS256.Decrypt(Span<byte> plaintext, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ReadOnlySpan<byte> associatedData = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`plaintext` has a length not equal to `ciphertext.Length - TagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length less than `TagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid authentication tag for the given inputs.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int NonceSize = 32;
public const int TagSize = 32;
```

## Notes

{% hint style="danger" %}
If you intend to feed **multiple, variable-length** inputs into the associated data, beware of [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Please read the [Concat](../advanced/concat.md) page for more information.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It can either be [randomly generated](../random-data.md#fill) or the output of a [KDF](../key-derivation.md). Furthermore, it **SHOULD** be rotated periodically (e.g. a different key per file).
{% endhint %}

{% hint style="warning" %}
Encrypting data in 16-64 KiB chunks instead of as a single plaintext message is **RECOMMENDED** to keep memory usage low and detect corrupted chunks early. Unfortunately, it is difficult to get right. You **MUST** ensure that chunks cannot be:

1. Truncated
2. Removed
3. Reordered
4. Duplicated

1 and 2 can be accomplished by including the length of all the ciphertext chunks added together in the associated data of the first chunk. Alternatively, you can use the [STREAM](https://eprint.iacr.org/2015/189) construction.

3 and 4 can be resolved by using a [counter](../constant-time.md#increment) nonce or by including the previous tag in the associated data of the next chunk.

If decryption fails midway through a stream due to tampering or corruption, erase the previous plaintext outputs from memory and/or disk and throw an error.
{% endhint %}

{% hint style="warning" %}
As a _general_ rule, avoid compression before encryption. It can [leak information](https://iacr.org/archive/fse2002/23650264/23650264.pdf) and has been the cause of [several attacks](https://crypto.stackexchange.com/a/38055).
{% endhint %}

{% hint style="warning" %}
To make AEGIS-256 [fully committing](https://eprint.iacr.org/2022/1260), you can [hash](../hashing.md) the associated data using a cryptographic hash function with a minimum of 128-bit preimage resistance and use the output as the AEAD associated data parameter.
{% endhint %}

{% hint style="success" %}
Current popular AEAD schemes like AES-GCM and ChaCha20-Poly1305 are [non-committing](https://eprint.iacr.org/2020/1491). By contrast, AEGIS-256 is believed to be [key committing](https://eprint.iacr.org/2022/268) (but not [fully committing](https://eprint.iacr.org/2022/1260)). This prevents [attacks](https://eprint.iacr.org/2023/526) when the adversary can choose the key and/or nonce (but [not](https://eprint.iacr.org/2023/1495) when they can choose the associated data). For example, it is computationally infeasible to output an AEGIS-256 ciphertext that can be decrypted without an authentication error using a different key.
{% endhint %}

{% hint style="success" %}
AEGIS-256 is [significantly faster](https://eprint.iacr.org/2023/523) than (X)ChaCha20-Poly1305 and AES-GCM when there is AES hardware support, which is available on most modern x64/ARM64 CPUs.

Without hardware support, (X)ChaCha20-Poly1305 is faster, and the AEGIS-256 implementation may be vulnerable to side-channel attacks.
{% endhint %}

{% hint style="success" %}
AEGIS-256 can be used as a [MAC](../message-authentication.md) by encrypting with the message as the associated data and an empty plaintext, resulting in just a tag. However, it **MUST NOT** be used as a hash function (e.g. without a secret key or for key derivation).
{% endhint %}

{% hint style="info" %}
AEGIS-256 was [originally](https://competitions.cr.yp.to/round3/aegisv11.pdf) specified to use a 128-bit tag. This is currently not supported in libsodium. Similarly, AEGIS-128 from the [CAESAR competition](https://competitions.cr.yp.to/caesar-submissions.html) is not supported nor part of the [Internet-Draft](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-aegis-aead).
{% endhint %}
