# Validation

## Purpose

The validation methods Geralt uses can be called to avoid ugly if statement validation at the top of functions. This is handy for [custom constructions](https://github.com/samuel-lucas6/kcChaCha20-Poly1305).

## Usage

### EqualToSize

Checks that an integer is equal to the valid size.

```csharp
Validation.EqualToSize(string paramName, int size, int validSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is not equal to `validSize`.

### SizeBetween

Checks that an integer is between the minimum and maximum size.

```csharp
Validation.SizeBetween(string paramName, int size, int minSize, int maxSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is less than `minSize` or greater than `maxSize`.

### NotLessThanMin

Checks that an integer is not less than the minimum size.

```csharp
Validation.NotLessThanMin(string paramName, int size, int minSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is less than `minSize`.

### NotGreaterThanMax

Checks that an integer is not greater than the maximum size.

```csharp
Validation.NotGreaterThanMax(string paramName, int size, int maxSize)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is greater than `maxSize`.

### MultipleOfSize

Checks that an integer is a multiple of another integer.

```csharp
Validation.MultipleOfSize(string paramName, int size, int multipleOf)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is less than or equal to zero or not a multiple of `multipleOf`.

### NotEmpty

Checks that the length of a span is not equal to zero.

```csharp
Validation.NotEmpty(string paramName, int size)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is equal to 0.

### GreaterThanZero

Checks that an integer is not less than or equal to zero.

```csharp
Validation.GreaterThanZero(string paramName, int size)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is less than or equal to 0.

### NotNull

Checks that an array is not null.

```csharp
Validation.NotNull<T>(string paramName, T value)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`value` is null.

### NotNullOrEmpty

Checks that a string is not null or empty.

```csharp
Validation.NotNullOrEmpty(string paramName, string value)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`value` is null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` has a length of 0.
