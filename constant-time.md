# Constant time

## Purpose

Whenever you interact with secrets and cryptographic parameters, you should use constant time functions to avoid leaking information via timing to an attacker. Such leaks can completely compromise security.

For example, you should increment counters/nonces, compare tags, compare retyped passwords, and so on using this class.

## Usage

### Equals

Returns whether two spans are equal in length and contain equal data.

```csharp
bool equal = ConstantTime.Equals(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a.Length` is equal to 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b.Length` is equal to 0.

### Increment

Increments a span counter in little-endian format.

```csharp
ConstantTime.Increment(Span<byte> buffer);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is equal to 0.

### Add

Fills a span with the sum of two spans in little-endian format.

```csharp
ConstantTime.Add(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is equal to 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a.Length` is equal to 0 or not equal to `buffer.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b.Length` is equal to 0 or not equal to `a.Length`.

### Subtract

Fills a span with the result of subtracting the second span from the first span in little-endian format.

```csharp
ConstantTime.Subtract(Span<byte> buffer, ReadOnlySpan<byte> a, ReadOnlySpan<byte> b);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`buffer.Length` is equal to 0.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a.Length` is equal to 0 or not equal to `buffer.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b.Length` is equal to 0 or not equal to `a.Length`.

### IsLessThan

Returns whether the contents of the first span is less than the second span in little-endian format.

```csharp
bool lessThan = ConstantTime.IsLessThan(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a.Length` is equal to 0 or not equal to `b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b.Length` is equal to 0.

### IsGreaterThan

Returns whether the contents of the first span is greater than the second span in little-endian format.

```csharp
bool greaterThan = ConstantTime.IsGreaterThan(ReadOnlySpan<byte> a, ReadOnlySpan<byte> b);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`a.Length` is equal to 0 or not equal to `b.Length`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`b.Length` is equal to 0.

### IsAllZeros

Returns whether a span only contains zeros.

```csharp
bool allZeros = ConstantTime.IsAllZeros(ReadOnlySpan<byte> buffer);
```

#### Exceptions

N/A

## Notes

{% hint style="danger" %}
[Tags](message-authentication.md) **MUST** be compared in constant time using `ConstantTime.Equals()` to prevent [forgery](https://crypto.stackexchange.com/a/101635). The `VerifyTag()` and `FinalizeAndVerify()` functions do this for you.
{% endhint %}

{% hint style="success" %}
These constant time functions can also be used for non-secret values. If in doubt, use constant time.
{% endhint %}
