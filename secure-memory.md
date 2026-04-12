# Secure memory

## Purpose

If an attacker can retrieve sensitive data, like keys, passwords, and plaintexts, they can effectively bypass the associated cryptography. Therefore, it is recommended to limit the exposure of and protect such sensitive data in memory, reducing the likelihood of it being accessible to attackers.

Here are several steps you should take as a developer:

1. Use byte or char arrays over strings. Strings are immutable, meaning they can't reliably be erased and copies may be made. [SecureString](https://learn.microsoft.com/en-gb/dotnet/api/system.security.securestring) is also [not worth using](https://github.com/dotnet/platform-compat/blob/master/docs/DE0001.md).
2. Use [stackalloc](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/stackalloc) or [GC.AllocateArray\<T>()](https://learn.microsoft.com/en-us/dotnet/api/system.gc.allocatearray) with `pinned: true` for allocations. This prevents .NET from copying the memory around, allowing proper erasure.
3. Use [ZeroMemory()](secure-memory.md#zeromemory) to wipe arrays in a manner that won't be optimised away by the compiler. This should be done as soon as the array is no longer needed and when an exception occurs. For example, you can call this method in a [try-finally](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements) statement.
4. Use [guarded heap allocations](secure-memory.md#guardedheapallocation), which perform memory locking and limit access to the memory at an operating system level. This protects against data being swapped to disk, buffer overflows/underflows, and other processes accessing the data.
5. Use a library like [memguard](https://github.com/awnumar/memguard) (a Go library, but a C# equivalent may be made) that [encrypts memory](https://spacetime.dev/encrypting-secrets-in-memory) whilst using guarded heap allocations. This does all of the above whilst protecting against things like [cold boot attacks](https://en.wikipedia.org/wiki/Cold_boot_attack) and [speculative execution vulnerabilities](https://en.wikipedia.org/wiki/Speculative_execution#Security_vulnerabilities).
6. Support using hardware that stores secrets and performs cryptographic operations on your behalf, like [YubiKeys](https://docs.yubico.com/yesdk/users-manual/getting-started/what-is-a-yubikey.html).

{% hint style="success" %}
The first three bullets are the most important.
{% endhint %}

## Usage

### ZeroMemory

Overwrites a byte array with zeros.

```csharp
SecureMemory.ZeroMemory(Span<byte> buffer);
```

{% hint style="warning" %}
`buffer` **MUST** be [pinned](secure-memory.md#purpose) for this method to work properly.
{% endhint %}

#### Exceptions

N/A

### ZeroMemory

Overwrites a char array or string with zeros.

```csharp
SecureMemory.ZeroMemory(ReadOnlySpan<char> buffer);
```

{% hint style="warning" %}
`buffer` **MUST** be [pinned](secure-memory.md#purpose) for this method to work properly.

Also, calling this function on a string [converted to a span](https://learn.microsoft.com/en-us/dotnet/api/system.memoryextensions.asspan#system-memoryextensions-asspan\(system-string\)) may crash the .NET runtime in the future. However, it doesn't currently based on my testing.
{% endhint %}

#### Exceptions

N/A

### GuardedHeapAllocation

Provides access to guarded heap allocations, which can be used instead of regular allocations for additional security. However, there is a performance penalty, and this functionality **SHOULD NOT** be used for large amounts of variables/data due to system limits.

```csharp
// Instead of Span<byte> key = stackalloc byte[32];
using var key = new GuardedHeapAllocation(int size);

// Access the memory
SecureRandom.Fill(key.AsSpan());

// Make the memory inaccessible
key.NoAccess();

// Make the memory read-only
key.ReadOnly();

// Make the memory readable and writable
key.ReadWrite();

// Destroy the secret (done automatically with a using statement)
key.Dispose();
```

{% hint style="danger" %}
Do **NOT** assign the `AsSpan()` return value to a span variable. <mark style="color:yellow;">This is a good way to crash your application</mark>. Instead, call `AsSpan()` every time you want to access the memory.

Similarly, do **NOT** access the memory after calling `NoAccess()` or try to write to the memory after calling `ReadOnly()`. <mark style="color:yellow;">This will immediately crash your application</mark>.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`size` is less than 1 or greater than `MaxSize`.

[InsufficientMemoryException](https://learn.microsoft.com/en-us/dotnet/api/system.insufficientmemoryexception)

Insufficient memory for guarded heap allocation.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Error marking memory as inaccessible.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Error marking memory as read-only.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Error marking memory as readable and writable.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

## Constants

These are used for validation and/or save you from defining your own constants.

```csharp
// GuardedHeapAllocation class
public static readonly int MaxSize = Environment.SystemPageSize - 16;
```

## Notes

Guarded heap allocations have the following memory layout, with a few caveats:

* The stored size in the first guard page refers to the canary plus data size rounded up to a multiple of the page size.
* The length of the stored size in the first guard page depends on the platform, as does the page size.
* The canary plus data does not necessarily take up a whole page; there may be padding at the beginning (unused memory) and multiple pages (if the data is large enough).
* The canary is initialized with random bytes, and data is initialized with `0xdb` bytes to help catch bugs due to uninitialized data.
* If access protection is not supported on the platform, the last page contains a canary at the beginning, meaning there can be two canaries with the same value surrounding the data. In this scenario, access to guard pages is not restricted.

<figure><img src=".gitbook/assets/Guarded heap allocations v3.svg" alt=""><figcaption><p>Guarded heap allocation layout</p></figcaption></figure>

{% hint style="warning" %}
Guarded heap allocations are not as fast as ordinary allocations and use up a lot more memory. Additionally, there are system limits on how much memory can be locked by a process. Therefore, use them sparingly for small amounts of data and test your application thoroughly.
{% endhint %}

{% hint style="warning" %}
Memory locking is not guaranteed with guarded heap allocations. For example, it may fail or not be supported on the system, in which case the function continues with no error.
{% endhint %}

{% hint style="success" %}
It is **RECOMMENDED** to use a separate guarded heap allocation for each secret. This is less error-prone whilst offering greater protection and access control.
{% endhint %}

{% hint style="success" %}
As well as taking these steps as a developer, you can recommend that users of your application do the following:

* Limit the number of installed/running applications on their machine.
* Harden their machine against malware (e.g., allowlisting, a local firewall, etc).
* Encrypt their OS drive.
* Disable hibernation.
* Disable memory dumps.
* Disable swap files/the swap partition.
{% endhint %}

{% hint style="info" %}
Memory locking uses [VirtualLock()](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtuallock) on Windows and [mlock()](https://linux.die.net/man/2/mlock) with [madvise()](https://linux.die.net/man/2/madvise) and `MADV_DONTDUMP` on Unix.

Guarded heap allocations require 3 or 4 extra pages of memory over a typical allocation. Allocations are done with [VirtualAlloc()](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) on Windows and [mmap()](https://linux.die.net/man/2/mmap) with `MAP_NOCORE` or [posix\_memalign()](https://linux.die.net/man/3/posix_memalign) on Unix. [VirtualProtect()](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect) on Windows and [mprotect()](https://linux.die.net/man/2/mprotect) on Unix are used for the memory access restrictions.

The protection on Unix is [likely superior](https://github.com/awnumar/memguard/issues/123) to on Windows.
{% endhint %}
