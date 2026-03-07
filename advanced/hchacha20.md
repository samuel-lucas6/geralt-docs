# HChaCha20

## Purpose

[HChaCha20](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-xchacha#section-2.2) is a pseudorandom function (PRF) based on [ChaCha20](https://cr.yp.to/chacha/chacha-20080128.pdf) and [HSalsa20](https://cr.yp.to/snuffle/xsalsa-20110204.pdf). It takes a 512-bit input and produces a 256-bit output. This makes it suitable for fast key derivation from **high-entropy** input keying material, as done for [XChaCha20](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-xchacha#section-2.3).

{% hint style="warning" %}
You probably want [BLAKE2b](../key-derivation.md) instead because it is much more flexible. [Argon2id](../password-hashing.md) **MUST** be used for password-based key derivation.
{% endhint %}

## Usage

### DeriveKey

Fills a span with output keying material derived from **high-entropy** input keying material, a nonce, and an optional personalization constant for domain separation.

```csharp
HChaCha20.DeriveKey(Span<byte> outputKeyingMaterial, ReadOnlySpan<byte> inputKeyingMaterial, ReadOnlySpan<byte> nonce, ReadOnlySpan<byte> personalization = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`outputKeyingMaterial` has a length not equal to `OutputSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`inputKeyingMaterial` has a length not equal to `KeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`nonce` has a length not equal to `NonceSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`personalization` has a length greater than 0 but not equal to `PersonalSize`.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int OutputSize = 32;
public const int KeySize = 32;
public const int NonceSize = 16;
public const int PersonalSize = 16;
```

## Notes

{% hint style="danger" %}
HChaCha20 is **NOT** a general-purpose hash function. Use [BLAKE2b](../hashing.md).
{% endhint %}

{% hint style="danger" %}
The input keying material **MUST** be a uniformly random key, **NOT** a password, public key, or X25519 shared secret.
{% endhint %}
