# Password hashing

## Purpose

[Argon2id](https://www.rfc-editor.org/rfc/rfc9106.html) is a memory-hard password hashing and password-based key derivation function (KDF). It takes the following parameters:

* A password.
* A 128-bit [random](random-data.md#fill) salt.
* An iteration count.
* A memory size in bytes.

{% hint style="success" %}
1. Set the iteration count to 3.
2. Set the memory size as high as possible (minimum of 64 MiB) for a reasonable delay (e.g. 100 ms to 1 sec) on the type of device your application will run on.
3. If the delay is lower than you would like, increase the iterations.

See the [Notes](password-hashing.md#notes) for some example parameters.
{% endhint %}

## Usage

### DeriveKey

Fills a span with output keying material computed from a password, a [random](random-data.md#fill) salt, an iteration count, and a memory size in bytes.

```csharp
Argon2id.DeriveKey(Span<byte> outputKeyingMaterial, ReadOnlySpan<byte> password, ReadOnlySpan<byte> salt, int iterations, int memorySize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`outputKeyingMaterial` has a length less than `MinKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`salt` has a length not equal to `SaltSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`iterations` is less than `MinIterations`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`memorySize` is less than `MinMemorySize`.

[InsufficientMemoryException](https://docs.microsoft.com/en-us/dotnet/api/system.insufficientmemoryexception)

Insufficient memory to perform key derivation.

### ComputeHash

Fills a span with an encoded password hash computed from a password, a randomly generated salt, an iteration count, and a memory size in bytes.

```csharp
Argon2id.ComputeHash(Span<byte> hash, ReadOnlySpan<byte> password, int iterations, int memorySize)
```

{% hint style="success" %}
`hash` must be a fixed length due to libsodium's API, which pads the potentially variable-length output with null characters.

You can convert the hash into a string for storage in a database using [Encoding.UTF8.GetString()](https://learn.microsoft.com/en-us/dotnet/api/system.text.encoding.getstring). Any null characters at the end can either be left alone or removed. Only this hash needs to be stored as the cost parameters and salt are encoded.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length not equal to `MaxHashSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`iterations` is less than `MinIterations`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`memorySize` is less than `MinMemorySize`.

[InsufficientMemoryException](https://docs.microsoft.com/en-us/dotnet/api/system.insufficientmemoryexception)

Insufficient memory to perform password hashing.

### VerifyHash

Verifies that an encoded password hash is correct for a given password. It returns `true` if the hash is valid and `false` otherwise.

```csharp
Argon2id.VerifyHash(ReadOnlySpan<byte> hash, ReadOnlySpan<byte> password)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length less than `MinHashSize` or greater than `MaxHashSize`.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid encoded password hash prefix.

### NeedsRehash

Determines if an encoded password hash matches the expected iteration count and memory size. It returns `true` if the hash does not match and `false` if the hash matches.

```csharp
Argon2id.NeedsRehash(ReadOnlySpan<byte> hash, int iterations, int memorySize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`hash` has a length less than `MinHashSize` or greater than `MaxHashSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`iterations` is less than `MinIterations`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`memorySize` is less than `MinMemorySize`.

[FormatException](https://docs.microsoft.com/en-us/dotnet/api/system.formatexception)

Invalid encoded password hash.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int SaltSize = 16;
public const int MinKeySize = 16;
public const int MinIterations = 1;
public const int MinMemorySize = 8192;
public const int MinHashSize = 93;
public const int MaxHashSize = 128;
```

## Notes

{% hint style="info" %}
The best defence against password cracking will always be to use strong passwords. For example, [diceware](https://www.eff.org/dice) with 6+ words.
{% endhint %}

{% hint style="success" %}
* Interactive scenario (e.g. online login): 50-250 ms.
* Semi-interactive scenario (e.g. file encryption): 250-1000 ms.
* Non-interactive (e.g. disk encryption): 1000-5000 ms.
{% endhint %}

Here are some example parameters for different scenarios:

|             Source            | Iterations | Memory (bytes) | \*Delay (ms) |
| :---------------------------: | :--------: | :------------: | :----------: |
|    libsodium's interactive    |      2     |    67108864​   |      51      |
| RFC second recommended option |      3     |    67108864​   |      72      |
|      libsodium's moderate     |      3     |    268435456   |      314     |
|     libsodium's sensitive     |      4     |   1073741824   |     1745     |

**\*These delays are for my desktop (a gaming PC)**. You should perform benchmarks on a typical device for your application using [BenchmarkArgon2.NET](https://github.com/samuel-lucas6/benchmark-argon2-dotnet).

{% hint style="success" %}
More memory is better than more iterations. However, you will need to increase the iterations in most cases because there should be a limit on how much memory your application uses.
{% endhint %}

{% hint style="warning" %}
Too high of an iteration count/memory size on a server could lead to denial-of-service (DoS) attacks. You can do client-side password hashing as well as server-side password hashing to help, sometimes called [server relief](https://doc.libsodium.org/password_hashing#server-relief).
{% endhint %}

{% hint style="info" %}
The parallelism is always 1 for deriving keys/hashes in libsodium. However, hashes with a parallelism greater than 1 can be verified.
{% endhint %}

{% hint style="info" %}
Libsodium also supports Argon2i, which is more side-channel resistant but less GPU resistant. However, Geralt only supports Argon2id because it is the [mandatory](https://www.rfc-editor.org/rfc/rfc9106.html#name-introduction) and [recommended](https://www.rfc-editor.org/rfc/rfc9106.html#name-recommendations) variant in the RFC plus there are [attacks](https://en.wikipedia.org/wiki/Argon2#Cryptanalysis) against Argon2i.
{% endhint %}
