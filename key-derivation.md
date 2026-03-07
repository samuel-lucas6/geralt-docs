# Key derivation

## Purpose

[BLAKE2b](https://datatracker.ietf.org/doc/html/rfc7693) can be used as a key derivation function (KDF) for **high-entropy** keys. It takes the following parameters to produce 256 to 512 bits of output keying material:

* 256 to 512 bits of input keying material (e.g. a shared secret).
* A 128-bit personalization constant (e.g. an application/protocol name).
* A 128-bit salt (e.g. a [counter](constant-time.md#increment) or [random](random-data.md#fill) data).
* Optional contextual info of any length (e.g. an explanation of what the key will be used for).

This allows you to derive new, distinct keys from a **high-entropy** master key. For example, separate keys for encryption and authentication with [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3) by changing the personalization constant, salt, and/or info.

{% hint style="danger" %}
BLAKE2b is **NOT** suitable for deriving keys from passwords. Use [Argon2id](password-hashing.md) instead.
{% endhint %}

{% hint style="success" %}
256-bit keys are recommended. Larger keys are unnecessary.
{% endhint %}

## Usage

### DeriveKey

Fills a span with output keying material computed from input keying material, a personalization constant, a salt, and optional additional contextual info.

```csharp
BLAKE2b.DeriveKey(Span<byte> outputKeyingMaterial, ReadOnlySpan<byte> inputKeyingMaterial, ReadOnlySpan<byte> personalization, ReadOnlySpan<byte> salt = default, ReadOnlySpan<byte> info = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`outputKeyingMaterial` has a length less than `MinKeySize` or greater than `MaxKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`inputKeyingMaterial` has a length less than `MinKeySize` or greater than `MaxKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`personalization` has a length not equal to `PersonalSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`salt` has a length greater than 0 but not equal to `SaltSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The key could not be derived.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int SaltSize = 16;
public const int PersonalSize = 16;
public const int MinKeySize = 32;
public const int MaxKeySize = 64;
```

## Notes

{% hint style="danger" %}
The input keying material **MUST** be high in entropy (e.g. a shared secret).
{% endhint %}

{% hint style="danger" %}
Do **NOT** use the same output keying material for multiple purposes (e.g. encryption and authentication). You should derive separate keys using the same input keying material and personalization but different salts and/or info.
{% endhint %}

{% hint style="danger" %}
If you intend to feed **multiple variable-length** inputs into the info, beware of [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Please read the [Concat](advanced/concat.md) page for more information.
{% endhint %}
