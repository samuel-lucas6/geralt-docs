# Message authentication

## Purpose

[BLAKE2b](https://datatracker.ietf.org/doc/html/rfc7693) is a cryptographic hash function and message authentication code (MAC). As a MAC, it takes a message of any size and a 256-bit to 512-bit key and produces a 128-bit to 512-bit tag.

This tag allows you to verify that a message has not been tampered with. A change to the message or the use of a different key will result in a different tag, at which point you should throw an error.

{% hint style="warning" %}
A tag size of at least 256 bits is **strongly recommended** to obtain [committing security](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html), which requires collision resistance.
{% endhint %}

{% hint style="success" %}
A 256-bit key is recommended regardless of the tag size. Larger keys are unnecessary.
{% endhint %}

## Usage

### ComputeTag

Fills a span with a tag computed from a message and a key.

```csharp
BLAKE2b.ComputeTag(Span<byte> tag, ReadOnlySpan<byte> message, ReadOnlySpan<byte> key)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`tag` has a length less than `MinTagSize` or greater than `MaxTagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length less than `MinKeySize` or greater than `MaxKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The tag could not be computed.

### VerifyTag

Verifies that a tag is correct in **constant time** for a given message and key. It returns `true` if the tag is valid and `false` otherwise.

```csharp
BLAKE2b.VerifyTag(ReadOnlySpan<byte> tag, ReadOnlySpan<byte> message, ReadOnlySpan<byte> key)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`tag` has a length less than `MinTagSize` or greater than `MaxTagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length less than `MinKeySize` or greater than `MaxKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The tag could not be recomputed.

### IncrementalBLAKE2b

Provides support for computing a tag from several messages and a key.

```csharp
using var blake2b = new IncrementalBLAKE2b(int hashSize, ReadOnlySpan<byte> key);
blake2b.Update(ReadOnlySpan<byte> message1);
blake2b.Update(ReadOnlySpan<byte> message2);
// Compute
blake2b.Finalize(Span<byte> tag1);
// Or verify
bool valid = blake2b.FinalizeAndVerify(ReadOnlySpan<byte> tag1);

// Avoid another using statement
blake2b.Reinitialize(int hashSize, ReadOnlySpan<byte> key);
// Cache the state
blake2b.CacheState();
blake2b.Update(ReadOnlySpan<byte> message3);
blake2b.Finalize(Span<byte> tag2);

// Restore the cached state
blake2b.RestoreCachedState();
blake2b.Update(ReadOnlySpan<byte> message3);
// tag3 == tag2
blake2b.Finalize(Span<byte> tag3);
```

{% hint style="warning" %}
`CacheState()` can only cache the state once. Each subsequent call will overwrite the previously cached state. See the [Notes](message-authentication.md#notes) for when this method should be used.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hashSize` is less than `MinHashSize` or greater than `MaxHashSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length less than `MinKeySize` or greater than `MaxKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length not equal to `hashSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The tag could not be computed.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot update after finalizing or finalize twice (without reinitializing or restoring a cached state).

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot cache the state after finalizing (without reinitializing).

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot restore the state when it has not been cached.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int TagSize = 32;
public const int KeySize = 32;
public const int MinTagSize = 16;
public const int MaxTagSize = 64;
public const int MinKeySize = 16;
public const int MaxKeySize = 64;
public const int BlockSize = 128;
```

## Notes

{% hint style="danger" %}
Tags **MUST** be compared in constant time to avoid leaking information, so use the `VerifyTag()` or `FinalizeAndVerify()` function.​
{% endhint %}

{% hint style="danger" %}
With [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3), you **MUST** include the nonce in the message when computing the tag.
{% endhint %}

{% hint style="danger" %}
If you intend to feed **multiple variable-length** inputs into the message, beware of [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Please read the [Concat](advanced/concat.md) page for more information.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It should be the output of a [KDF](key-derivation.md), **NOT** randomly generated.
{% endhint %}

{% hint style="danger" %}
Do **NOT** use the same key for multiple purposes (e.g., encryption and authentication). You should [derive](key-derivation.md) separate keys using the same input keying material and personalisation but different salts and/or info.
{% endhint %}

{% hint style="success" %}
The same key can be reused for multiple messages, but it is good practice to [derive](key-derivation.md) a unique key each time.
{% endhint %}

{% hint style="success" %}
If you are making multiple calls to `IncrementalBLAKE2b` with unchanging/static data at the beginning (e.g., the same key), you can cache the state to improve performance. This allows you to only process this data once and can help you quickly zero the key from memory.
{% endhint %}

{% hint style="info" %}
The security level of BLAKE2b against a generic attack on hash-based MACs is 1/2 the output length (e.g., 128-bit security for a 256-bit tag).​ However, the security level is equal to the output length for typical attacks against MACs (e.g., 256-bit security for a 256-bit tag). Both types of attacks are completely impractical.
{% endhint %}
