# ChaCha20-Poly1305

## Purpose

[ChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/rfc8439#section-2.8) is an authenticated encryption with associated data (AEAD) scheme. It encrypts a plaintext message using a 256-bit key and 96-bit nonce (**number used only once**) before computing a tag over the ciphertext and associated data.

The associated data is useful for authenticating file headers, version numbers, timestamps, counters, and so on. It can be used to prevent [confused deputy attacks](https://cloud.google.com/kms/docs/additional-authenticated-data#confused_deputy_attack_example) and [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). It is not encrypted nor part of the ciphertext. It must be reproduceable or stored somewhere for decryption to be possible.

For decryption, the tag is first verified for the given inputs, which detects tampering and incorrect parameters. If verification fails, an error is returned. Otherwise, the ciphertext is decrypted and plaintext is returned.

{% hint style="warning" %}
Use [XChaCha20-Poly1305](xchacha20-poly1305.md) if you want random nonces with the same key.
{% endhint %}

{% hint style="danger" %}
For encryption, the nonce **MUST NOT** be repeated or reused with the same key. You **MUST** [increment](../constant-time.md#increment) the nonce for each plaintext message encrypted using the same key.
{% endhint %}

## Usage

### Encrypt

Fills a span with ciphertext and an appended tag computed from a plaintext message, nonce, key, and optional associated data.

```csharp
ChaCha20Poly1305.Encrypt(Span<byte> ciphertext, ReadOnlySpan<byte> plaintext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ReadOnlySpan<byte> associatedData = default)
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
ChaCha20Poly1305.Decrypt(Span<byte> plaintext, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ReadOnlySpan<byte> associatedData = default)
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
public const int NonceSize = 12;
public const int TagSize = 16;
```

## Notes

{% hint style="danger" %}
If you intend to feed **multiple variable-length** inputs into the associated data, beware of [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Please read the [Concat](../advanced/concat.md) page for more information.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It can either be [randomly generated](../random-data.md#fill) or the output of a [KDF](../key-derivation.md). Furthermore, it **SHOULD** be rotated periodically (e.g., a different key per file).
{% endhint %}

{% hint style="warning" %}
Encrypting data in 16-64 KiB chunks instead of as a single plaintext message is **recommended** to keep memory usage low and detect corrupted chunks early. Unfortunately, it is difficult to get right. You **MUST** ensure that chunks cannot be:

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
ChaCha20-Poly1305 is **NOT** [key- or message-committing](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#name-introduction).

1. A ciphertext message can be decrypted under multiple keys without an error.
2. An attacker who knows the key can find different messages that lead to the same tag.

This can enable [attacks](https://youtu.be/dZqEtrLh9aM) in scenarios where keys can be adversarial. For example, when an attacker can submit a ciphertext encrypted using a password to a server that knows the encryption key (an oracle).

The best fix is to switch to [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3) as outlined in that link.
{% endhint %}
