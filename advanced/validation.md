# Validation

## Purpose

The validation methods that Geralt uses can be called to avoid ugly if statement validation at the top of functions. This is handy for [custom constructions](https://github.com/samuel-lucas6/kcChaCha20-Poly1305).

## Usage

### EqualTo

Checks that an integer is equal to a required value.

```csharp
Validation.EqualTo<T>(string paramName, T value, T required) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be equal to `required`.

### BetweenOrEqualTo

Checks that an integer is between a minimum and maximum value (inclusive).

```csharp
Validation.BetweenOrEqualTo<T>(string paramName, T value, T min, T max) where T : IBinaryInteger<T>)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be between `min` and `max` (inclusive).

### GreaterThan

Checks that an integer is greater than a minimum value.

```csharp
Validation.GreaterThan<T>(string paramName, T value, T min) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be greater than `min`.

### GreaterThanOrEqualTo

Checks that an integer is greater than or equal to a minimum value.

```csharp
Validation.GreaterThanOrEqualTo<T>(string paramName, T value, T min) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be greater than or equal to `min`.

### LessThanOrEqualTo

Checks that an integer is less than or equal to a maximum value.

```csharp
Validation.LessThanOrEqualTo<T>(string paramName, T value, T max) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be less than or equal to `max`.

### MultipleOf

Checks that an integer is a multiple of another integer.

```csharp
Validation.MultipleOf<T>(string paramName, T value, T multipleOf) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be a multiple of `multipleOf` (and not less than or equal to 0).

### NotEmpty

Checks that the length of a span is not equal to zero.

```csharp
Validation.NotEmpty(string paramName, int size)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is equal to 0.

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
