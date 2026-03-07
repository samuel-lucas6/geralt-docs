# XChaCha20

## Purpose

[XChaCha20](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-xchacha#section-2.3) is an **unauthenticated** stream cipher constructed using [ChaCha20](chacha20.md) and [HChaCha20](hchacha20.md). It takes a 256-bit key and 192-bit nonce (**number used only once**) to encrypt/decrypt a message.

The 64-bit internal counter can be changed from the default of 0 to access any block without computing previous ones. However, it should generally not be touched.

{% hint style="warning" %}
You probably want [XChaCha20-Poly1305](../authenticated-encryption/xchacha20-poly1305.md) instead, which also ensures a message has not been tampered with. This class **MUST** only be used for custom constructions (e.g. [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3)).
{% endhint %}

{% hint style="danger" %}
The nonce **MUST NOT** be repeated or reused with the same key. You **MUST** [increment](../constant-time.md#increment) or [randomly generate](../random-data.md#fill) the nonce for each plaintext message encrypted using the same key.
{% endhint %}

## Usage

### Fill

Fills a span with pseudorandom bytes computed from a nonce and key. This can be used to compute the Poly1305 key for constructing [XChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-xchacha).

```csharp
XChaCha20.Fill(Span<byte> buffer, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error computing pseudorandom bytes.

### Encrypt

Fills a span with ciphertext computed from a plaintext message, nonce, and key.

```csharp
XChaCha20.Encrypt(Span<byte> ciphertext, ReadOnlySpan<byte> plaintext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ulong counter = 0)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `plaintext.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Encryption failed.

### Decrypt

Fills a span with plaintext computed from a ciphertext message, nonce, and key.

```csharp
XChaCha20.Decrypt(Span<byte> plaintext, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, ulong counter = 0)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`plaintext` has a length not equal to `ciphertext.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Decryption failed.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int NonceSize = 24;
public const int BlockSize = 64;
```

## Notes

{% hint style="danger" %}
This is **NOT** an authenticated encryption algorithm. This class **MUST** only be used if you know what you are doing and apply authentication [yourself](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#name-the-generalised-caead-schem) using keyed [BLAKE2b](../message-authentication.md) as a MAC.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It can either be [randomly generated](../random-data.md#fill) or the output of a [KDF](../key-derivation.md). Furthermore, it **SHOULD** be rotated periodically (e.g. a different key per file).
{% endhint %}

{% hint style="warning" %}
As a _general_ rule, avoid compression before encryption. It can [leak information](https://iacr.org/archive/fse2002/23650264/23650264.pdf) and has been the cause of [several attacks](https://crypto.stackexchange.com/a/38055).
{% endhint %}

{% hint style="warning" %}
Even with [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3), it is recommended to encrypt data in 16-64 KiB chunks instead of as a single plaintext message. Read the [XChaCha20-Poly1305 Notes](../authenticated-encryption/xchacha20-poly1305.md#notes) for more information.
{% endhint %}
