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

### Between

Checks that an integer is between a minimum and maximum value (exclusive).

```csharp
Validation.Between<T>(string paramName, T value, T min, T max) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be between `min` and `max` (exclusive).

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

### LessThan

Checks that an integer is less than a maximum value.

```csharp
Validation.LessThan<T>(string paramName, T value, T max) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`value` must be less than `max`.

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

Checks that an integer (e.g. the length of a span) is not equal to zero.

```csharp
Validation.NotEmpty<T>(string paramName, T size) where T : IBinaryInteger<T>
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` must not be equal to 0.

### NotNull

Checks that a parameter is not null.

```csharp
Validation.NotNull<T>(string paramName, T? value)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`value` must not be null.

### NotNullOrEmpty

Checks that an enumerable (e.g., array) is not null or empty.

```csharp
Validation.NotNullOrEmpty<T>(string paramName, IEnumerable<T?> enumerable)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`enumerable` must not be null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`enumerable` must not be empty.

### HasNoNullValues

Checks that an enumerable (e.g., array) is not null and has no null values.

```csharp
Validation.HasNoNullValues<T>(string paramName, IEnumerable<T?> enumerable)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`enumerable` must not be null.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`enumerable` must not contain any null values.

### HasNoNullOrEmptyValues

Checks that an enumerable (e.g., string array) is not null/empty and has no null/empty values.

```csharp
Validation.HasNoNullOrEmptyValues<T>(string paramName, IEnumerable<T?> enumerable)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`enumerable` must not be null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`enumerable` must not be empty.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`enumerable` must not contain any null or empty values.

### HasNoNullValues

Checks that a jagged enumerable (e.g., jagged array) is not null and has no null values.

```csharp
Validation.HasNoNullValues<T>(string paramName, IEnumerable<T[]?> jaggedEnumerable)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`jaggedEnumerable` must not be null.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`jaggedEnumerable` must not contain any null collections/values.

### HasNoNullOrEmptyValues

Checks that a jagged enumerable (e.g., jagged string array) is not null/empty and has no null/empty values.

```csharp
Validation.HasNoNullOrEmptyValues<T>(string paramName, IEnumerable<T[]?> jaggedEnumerable)
```

#### Exceptions

[ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception)

`jaggedEnumerable` must not be null.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`jaggedEnumerable` must not be empty.

[ArgumentException](https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception)

`jaggedEnumerable` must not contain any null or empty collections/values.
