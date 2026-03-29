# ChaCha20

## Purpose

[ChaCha20](https://datatracker.ietf.org/doc/html/rfc8439#section-2.4) is an **unauthenticated** stream cipher that uses a 256-bit key and 96-bit nonce (**number used only once**) to encrypt/decrypt a message.

The 32-bit internal counter can be changed from the default of 0 to 1 for constructing [ChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/rfc8439). Otherwise, it should generally be left alone.

{% hint style="warning" %}
You probably want [ChaCha20-Poly1305](../authenticated-encryption/chacha20-poly1305.md) instead, which also ensures a message has not been tampered with. This class **MUST** only be used for custom constructions (e.g., [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3)).
{% endhint %}

{% hint style="danger" %}
The nonce **MUST NOT** be repeated or reused with the same key. You **MUST** [increment](../constant-time.md#increment) the nonce for each plaintext message encrypted using the same key.
{% endhint %}

## Usage

### Fill

Fills a span with pseudorandom bytes computed from a nonce and key. This can be used to compute the Poly1305 key for constructing [ChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/rfc8439).

```csharp
ChaCha20.Fill(Span<byte> buffer, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key)
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
ChaCha20.Encrypt(Span<byte> ciphertext, ReadOnlySpan<byte> plaintext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, uint counter = 0)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `plaintext.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

Counter overflow prevented.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error encrypting plaintext.

### Decrypt

Fills a span with plaintext computed from a ciphertext message, nonce, and key.

```csharp
ChaCha20.Decrypt(Span<byte> plaintext, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> key, uint counter = 0)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`plaintext` has a length not equal to `ciphertext.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

Counter overflow prevented.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error decrypting ciphertext.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int NonceSize = 12;
public const int BlockSize = 64;
```

## Notes

{% hint style="danger" %}
This is **NOT** an authenticated encryption algorithm. This class **MUST** only be used if you know what you are doing and apply authentication [yourself](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#name-the-generalised-caead-schem) using keyed [BLAKE2b](../message-authentication.md) as a MAC.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It can either be [randomly generated](../random-data.md#fill) or the output of a [KDF](../key-derivation.md). Furthermore, it **SHOULD** be rotated periodically (e.g., a different key per file).
{% endhint %}

{% hint style="warning" %}
As a _general_ rule, avoid compression before encryption. It can [leak information](https://iacr.org/archive/fse2002/23650264/23650264.pdf) and has been the cause of [several attacks](https://crypto.stackexchange.com/a/38055).
{% endhint %}

{% hint style="warning" %}
Even with [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3), it is recommended to encrypt data in 16-64 KiB chunks instead of as a single plaintext message. Read the [ChaCha20-Poly1305 Notes](../authenticated-encryption/chacha20-poly1305.md#notes) for more information.
{% endhint %}
