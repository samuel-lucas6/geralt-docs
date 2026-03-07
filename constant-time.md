# Constant time

## Purpose

Whenever you interact with secrets and cryptographic parameters, you should use constant time functions to avoid leaking information via timing to an attacker. Such leaks can completely compromise security.

For example, you should increment counters/nonces, compare tags, compare passwords, and so on using this class.

## Usage

### Equals

Determines if two spans are equal in length and contain equal data. It returns `true` if so and `false` otherwise.

```csharp
ConstantTime.Equals(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length of 0.

### Increment

Increments a span counter.

```csharp
ConstantTime.Increment(Span<byte> buffer)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

### Add

Fills a span with the sum of two spans.

```csharp
ConstantTime.Add(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length of 0 or not equal to `buffer.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length of 0 or not equal to `a.Length`.

### Subtract

Fills a span with the result of subtracting the second span from the first span.

```csharp
ConstantTime.Subtract(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer` has a length of 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length of 0 or not equal to `buffer.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length of 0 or not equal to `a.Length`.

### IsLessThan

Determines if the contents of the first span is less than the second span. It returns `true` if so and `false` otherwise.

```csharp
ConstantTime.IsLessThan(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length of 0 or not equal to `b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length of 0.

### IsGreaterThan

Determines if the contents of the first span is greater than the second span. It returns `true` if so and `false` otherwise.

```csharp
ConstantTime.IsGreaterThan(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a` has a length of 0 or not equal to `b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b` has a length of 0.

### IsAllZeros

Determines if a span only contains zeros. It returns `true` if so and `false` otherwise.

```csharp
ConstantTime.IsAllZeros(ReadOnlySpan<byte> buffer)
```

## Notes

{% hint style="danger" %}
[Tags](message-authentication.md) **MUST** be compared in constant time using `ConstantTime.Equals()`. The `VerifyTag()` and `FinalizeAndVerify()` functions do this for you.
{% endhint %}

{% hint style="success" %}
These constant time functions can also be used for non-secret values.
{% endhint %}

{% hint style="info" %}
All of these functions use a little-endian format.
{% endhint %}
