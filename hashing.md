# Hashing

## Purpose

[BLAKE2b](https://datatracker.ietf.org/doc/html/rfc7693) is a cryptographic hash function. It takes a message of any size and produces a 128-bit to 512-bit hash.

This hash acts as a fingerprint for the data. Hashes can be used to uniquely identify messages, detect corruption, detect duplicate data, and index data in a hash table.

However, **unkeyed** hashes do not provide [authentication](message-authentication.md) (e.g. for [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3)). Furthermore, they should be avoided for [key derivation](key-derivation.md). Use the linked APIs instead.

{% hint style="danger" %}
BLAKE2b is **NOT** suitable for hashing passwords. Use [Argon2id](password-hashing.md) instead.
{% endhint %}

{% hint style="danger" %}
A hash size of at least 256 bits is **strongly recommended** to obtain collision resistance.
{% endhint %}

## Usage

### ComputeHash

Fills a span with a hash computed from a message.

```csharp
BLAKE2b.ComputeHash(Span<byte> hash, ReadOnlySpan<byte> message)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length less than `MinHashSize` or greater than `MaxHashSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The hash could not be computed.

### IncrementalBLAKE2b

Provides support for computing a hash from several messages.

```csharp
using var blake2b = new IncrementalBLAKE2b(int hashSize);
blake2b.Update(ReadOnlySpan<byte> message1);
blake2b.Update(ReadOnlySpan<byte> message2);
blake2b.Finalize(Span<byte> hash1);

// Avoid another using statement
blake2b.Reinitialize(int hashSize);
blake2b.Update(ReadOnlySpan<byte> message3);
// Cache the state
blake2b.CacheState();
blake2b.Finalize(Span<byte> hash2);

// Restore the cached state
blake2b.RestoreCachedState();
// hash3 == hash2
blake2b.Finalize(Span<byte> hash3);
```

{% hint style="warning" %}
`CacheState()` can only cache the state once. Each subsequent call will overwrite the previously cached state. See the [Notes](hashing.md#notes) for when this method should be used.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hashSize` is less than `MinHashSize` or greater than `MaxHashSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length not equal to `hashSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The hash could not be computed.

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
public const int HashSize = 32;
public const int MinHashSize = 16;
public const int MaxHashSize = 64;
public const int BlockSize = 128;
public const int SaltSize = 16;
public const int PersonalizationSize = 16;
```

## Notes

{% hint style="danger" %}
Do **NOT** use `ComputeHash()` for key derivation. Read the [Key derivation](key-derivation.md) page instead.
{% endhint %}

{% hint style="warning" %}
Do **NOT** manually truncate a hash. Instead, specify the hash size you want directly. The hash size affects the output, which provides domain separation.
{% endhint %}

{% hint style="success" %}
Unlike older hash functions (e.g. MD5, SHA-1, SHA-256, and SHA-512), BLAKE2b is immune to [length extension attacks](https://en.wikipedia.org/wiki/Length_extension_attack).
{% endhint %}

{% hint style="success" %}
If you are making multiple calls to `IncrementalBLAKE2b` with unchanging/static data at the beginning, you can cache the state to improve performance. This allows you to only process this data once. It is more relevant in [message authentication](message-authentication.md) scenarios, as explained on that page.
{% endhint %}

{% hint style="info" %}
The security level of BLAKE2b is 1/2 the output length (e.g. 128-bit security for a 256-bit hash).​
{% endhint %}
