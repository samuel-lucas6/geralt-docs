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

### FillDeterministic

Fills a span with **deterministic** bytes indistinguishable from random without knowing the seed.

```csharp
SecureRandom.FillDeterministic(Span<byte> buffer, ReadOnlySpan<byte> seed)
```

{% hint style="warning" %}
This should be reserved for tests and custom constructions (e.g., an [XOF](https://github.com/jedisct1/libsodium/issues/623)).
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`seed` has a length not equal to `SeedSize`.

### GetInt32

Returns a random integer between 0 (inclusive) and the upper bound (exclusive).

```csharp
SecureRandom.GetInt32(int upperBound)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`upperBound` is less than `MinUpperBound`.

### GenerateString

Fills a span with a random string based on the provided character set.

```csharp
SecureRandom.GenerateString(Span<char> buffer, ReadOnlySpan<char> characterSet)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is less than `MinStringSize` or greater than `MaxStringSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`characterSet.Length` is less than `MinCharacterSetSize`.

### GetPassphraseBufferSize

Returns the required buffer size for `GeneratePassphrase()`. The length of the longest word in the wordlist being used must be known (see `LongestWordSize` for examples).

```csharp
SecureRandom.GetPassphraseBufferSize(int longestWord, int wordCount)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`longestWord` must be greater than or equal to `MinLongestWordSize` and less than or equal to `MaxLongestWordSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`wordCount` must be greater than or equal to `MinWordCount` and less than or equal to `MaxWordCount`.

### GeneratePassphrase

Fills a span with a random passphrase using the [EFF's long wordlist](https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt) (minus hyphenated words).

```csharp
SecureRandom.GeneratePassphrase(Span<char> buffer, out int passphraseSize, int wordCount, char separatorChar = '-', bool capitalize = false, bool includeNumber = false)
```

{% hint style="danger" %}
`buffer` must be sliced using `passphraseSize` to remove padding.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is not equal to `GetPassphraseBufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`wordCount` is less than `MinWordCount` or greater than `MaxWordCount`.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`separatorChar` must be a printable character.

### GeneratePassphrase

Fills a span with a random passphrase using a custom wordlist.

```csharp
SecureRandom.GeneratePassphrase(Span<char> buffer, out int passphraseSize, ReadOnlySpan<string> wordlist, int wordCount, char separatorChar = '-', bool capitalize = false, bool includeNumber = false)
```

{% hint style="danger" %}
`buffer` must be sliced using `passphraseSize` to remove padding.
{% endhint %}

{% hint style="success" %}
View the source code for `GetWordlist()` to see how to format a custom wordlist. It is recommended to stick to English wordlists because this is what has been tested.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is not equal to `GetPassphraseBufferSize()`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`wordCount` is less than `MinWordCount` or greater than `MaxWordCount`.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`separatorChar` must be a printable character.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`wordlist.Length` is less than `MinWordlistSize`.

[FormatException](https://learn.microsoft.com/en-us/dotnet/api/system.formatexception)

`wordlist` must not contain empty words and must only contain printable characters (excluding spaces).

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

The longest word in the wordlist must be greater than or equal to `MinLongestWordSize` and less than or equal to `MaxLongestWordSize`.

### GetWordlist

Returns the built-in [EFF long wordlist](https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt) (minus hyphenated words) formatted correctly for `GeneratePassphrase()`.

```csharp
SecureRandom.GetWordlist()
```

#### Exceptions

N/A

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int SeedSize = 32;
public const string AlphabeticChars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
public const string NumericChars = "0123456789";
public const string SymbolChars = "!\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~";
public const string AlphanumericChars = AlphabeticChars + NumericChars;
public const string AlphanumericSymbolChars = AlphanumericChars + SymbolChars;
public const int MinUpperBound = 2;
public const int MinStringSize = 8;
public const int MaxStringSize = 128;
public const int MinCharacterSetSize = MinUpperBound;
public const int MinLongestWordSize = 1;
public const int MaxLongestWordSize = 45;
public const int MinWordlistSize = MinUpperBound;
public const int MinWordCount = 4;
public const int MaxWordCount = 20;

public enum LongestWordSize
{
    EffLong = 9, // https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt
    EffShort1 = 5, // https://www.eff.org/files/2016/09/08/eff_short_wordlist_1.txt
    EffShort2 = 10, // https://www.eff.org/files/2016/09/08/eff_short_wordlist_2_0.txt
    Bip39 = 8, // https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt
    Slip39 = 8, // https://github.com/satoshilabs/slips/blob/master/slip-0039/wordlist.txt
    Monero = 12, // https://github.com/monero-project/monero/blob/master/src/mnemonics/english.h
    Diceware = 6 // https://theworld.com/~reinhold/diceware.html (both 7776 and 8192 words)
     // Other wordlists: https://gist.github.com/atoponce/95c4f36f2bc12ec13242a3ccc55023af
}
```

## Notes

{% hint style="danger" %}
If these functions are called inside a virtual machine (VM) that has had a snapshot restored, the same output may be produced.​
{% endhint %}

{% hint style="info" %}
The libsodium library uses `RtlGenRandom()` on Windows and `getrandom` or `/dev/urandom` on Linux and macOS to generate cryptographically secure random numbers non-deterministically. Deterministic generation is done using the IETF version of [ChaCha20](https://datatracker.ietf.org/doc/html/rfc8439#section-2.4).
{% endhint %}
