# Concat

## Purpose

Sometimes it is necessary to concatenate data together to form an input or output. This is represented via the `||` symbol in pseudocode.

For example, an AEAD ciphertext is usually a ciphertext concatenated with a tag.

{% hint style="danger" %}
Concatenation is often not as simple as it sounds. You **MUST** be careful to avoid [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/).

This is important for [hashing](../hashing.md), [message authentication](../message-authentication.md) (e.g. [Encrypt-then-MAC](https://samuellucas.com/draft-lucas-generalised-committing-aead/draft-lucas-generalised-committing-aead.html#section-3)), associated data in [AEADs](../authenticated-encryption/), and info in [key derivation](../key-derivation.md).

Two solutions are:

1. Only ever concatenate fixed-length inputs. With two inputs, only one needs to be fixed-length. With three inputs, only two need to be fixed-length.
2. For variable-length inputs, also concatenate the length of each input as a 64-bit unsigned integer ([ulong](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types)) [converted to bytes in little-endian](https://docs.microsoft.com/en-us/dotnet/api/system.buffers.binary.binaryprimitives.writeuint64littleendian?). See [this](https://github.com/samuel-lucas6/draft-lucas-generalised-committing-aead/blob/a8feb3e077a89ba6660b3cc21ca0b4f3780aeca8/reference-implementation/cAEAD/cAEAD/ChaCha20BLAKE2b.cs#L88) C# code example. This is done internally in AEADs like [ChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/rfc8439#section-2.8.1).
{% endhint %}

## Usage

### Concat

Fills a span with the concatenation of two spans.

```csharp
Spans.Concat(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `a.Length + b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length not equal to `buffer.Length - b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length not equal to `buffer.Length - a.Length`.

### Concat

Fills a span with the concatenation of three spans.

```csharp
Spans.Concat(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b, ReadOnlySpan<byte> c)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `a.Length + b.Length + c.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length not equal to `buffer.Length - b.Length - c.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length not equal to `buffer.Length - a.Length - c.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`c` has a length not equal to `buffer.Length - a.Length - b.Length`.

### Concat

Fills a span with the concatenation of four spans.

```csharp
Spans.Concat(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b, ReadOnlySpan<byte> c, ReadOnlySpan<byte> d)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `a.Length + b.Length + c.Length + d.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length not equal to `buffer.Length - b.Length - c.Length - d.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length not equal to `buffer.Length - a.Length - c.Length - d.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`c` has a length not equal to `buffer.Length - a.Length - b.Length - d.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`d` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length`.

### Concat

Fills a span with the concatenation of five spans.

```csharp
Spans.Concat(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b, ReadOnlySpan<byte> c, ReadOnlySpan<byte> d, ReadOnlySpan<byte> e)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `a.Length + b.Length + c.Length + d.Length + e.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length not equal to `buffer.Length - b.Length - c.Length - d.Length - e.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length not equal to `buffer.Length - a.Length - c.Length - d.Length - e.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`c` has a length not equal to `buffer.Length - a.Length - b.Length - d.Length - e.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`d` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length - e.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`e` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length - d.Length`.

### Concat

Fills a span with the concatenation of six spans.

```csharp
Spans.Concat(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b, ReadOnlySpan<byte> c, ReadOnlySpan<byte> d, ReadOnlySpan<byte> e, ReadOnlySpan<byte> f)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length not equal to `a.Length + b.Length + c.Length + d.Length + e.Length + f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length not equal to `buffer.Length - b.Length - c.Length - d.Length - e.Length - f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length not equal to `buffer.Length - a.Length - c.Length - d.Length - e.Length - f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`c` has a length not equal to `buffer.Length - a.Length - b.Length - d.Length - e.Length - f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`d` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length - e.Length - f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`e` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length - d.Length - f.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`f` has a length not equal to `buffer.Length - a.Length - b.Length - c.Length - d.Length - e.Length`.

## Notes

{% hint style="info" %}
There is unfortunately no `Concat()` function for arrays or spans in .NET, and there is not even a `Span<T>.CopyTo(Span<T> destination, int index)`. This class fills that gap.
{% endhint %}
