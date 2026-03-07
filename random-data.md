# Random data

## Purpose

This class produces **unpredictable, cryptographically secure** random numbers. Using a _predictable_ random number generator, such as [System.Random](https://docs.microsoft.com/en-us/dotnet/api/system.random), is **insecure**.

These functions should be used to randomly generate encryption keys, nonces, salts, seeds, integers, strings, and passphrases.

## Usage

### Fill

Fills a span with random bytes.

```csharp
SecureRandom.Fill(Span<byte> buffer)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

### GetInt32

Generates a random integer between 0 (inclusive) and the upper bound (exclusive).

```csharp
SecureRandom.GetInt32(int upperBound)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`upperBound` is less than `MinUpperBound`.

### GetString

Generates a random string of a given length. A custom character set can be provided, but several character sets are available via constants.

```csharp
SecureRandom.GetString(int length, string characterSet = AlphanumericChars)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`length` is less than `MinStringLength` or greater than `MaxStringLength`.

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`characterSet` is null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`characterSet` has a length of 0.

### GetPassphrase

Generates a random passphrase using the [EFF's long wordlist](https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt) (minus hyphenated words).

```csharp
SecureRandom.GetPassphrase(int wordCount, char separatorChar = '-', bool capitalize = false, bool includeNumber = false)
```

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`wordCount` is less than `MinWordCount` or greater than `MaxWordCount`.

### FillDeterministic

Fills a span with **deterministic** bytes indistinguishable from random without knowing the seed.

```csharp
SecureRandom.FillDeterministic(Span<byte> buffer, ReadOnlySpan<byte> seed)
```

{% hint style="warning" %}
This should be reserved for tests and custom constructions (e.g. an [XOF](https://github.com/jedisct1/libsodium/issues/623)).
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`seed` has a length not equal to `SeedSize`.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int SeedSize = 32;
public const string AlphabeticChars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
public const string NumericChars = "0123456789";
public const string SymbolChars = "!#$%&'()*+,-./:;<=>?@[]^_`{}~";
public const string AlphanumericChars = AlphabeticChars + NumericChars;
public const string AlphanumericSymbolChars = AlphanumericChars + SymbolChars;
public const int MinUpperBound = 2;
public const int MinStringLength = 8;
public const int MaxStringLength = 128;
public const int MinWordCount = 4;
public const int MaxWordCount = 20;
```

## Notes

{% hint style="danger" %}
If these functions are called inside a virtual machine (VM) which is snapshotted and restored, the same output may be produced.​
{% endhint %}

{% hint style="info" %}
The libsodium library uses `RtlGenRandom()` on Windows and `getrandom` or `/dev/urandom` on Linux and macOS to generate cryptographically secure random numbers non-deterministically. Deterministic generation is done using the IETF version of [ChaCha20](https://datatracker.ietf.org/doc/html/rfc8439#section-2.4).
{% endhint %}
