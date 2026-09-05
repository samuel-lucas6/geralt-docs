# Changelog

This page is a backup and merged version of the [GitHub release notes](https://github.com/samuel-lucas6/Geralt/releases), which document all notable changes to this project. The format is based on [Keep a Changelog](https://keepachangelog.com/en/2.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## v4.4.0 - 2026-07-26

#### Added

* The X-Wing post-quantum hybrid KEM (X25519 + ML-KEM-768).
* ML-KEM-768 (a non-hybrid, post-quantum KEM) for when you don't want to use X-Wing.
* Access to the Keccak-f\[1600] permutation for advanced users to be able to build custom constructions. Note that this API is more dangerous than the others if you aren't careful.

#### Changed

* Updated the test packages.
* Minor constants test improvements.

#### Unreleased

[KangarooTwelve](https://keccak.team/kangarootwelve.html) (KT128/KT256) is on the [libsodium roadmap](https://libsodium.gitbook.io/doc/roadmap). These XOFs will be added when they're made available and should be favoured over BLAKE2/BLAKE3.

I've decided against supporting SHA-3, SHAKE, and TurboSHAKE. The number of SHA-3 variants is silly, and these are all worse than KangarooTwelve. SHA-3 and SHAKE are also available in .NET's System.Security.Cryptography, albeit out of the box cross-platform support is a mess for modern .NET provided crypto.

## v4.3.0 - 2026-06-14

#### Changed

* libsodium defined constants are now checked against their libsodium function in tests.
* Updated the test packages.

#### Fixed/Security

{% hint style="success" %}
Thank you to Alexey Avramov ([@hakavlad](https://github.com/hakavlad)) for providing AI code audits of Geralt [v4.2.0](https://github.com/samuel-lucas6/Geralt/releases/tag/v4.2.0) over email, which identified several Informational-Medium issues. Most were repeat findings that are invalid or risk accepted. However, a few were new from rerunning the models, and one was a mistake in the previous release related to thread safety.

Gemini 3.1 Pro Preview bullet 5; Claude 4.7 Opus bullets 10 and 12; GPT-5.5 Pro bullets 1, 4, 5, and 6; and Claude Fable 5 bullets 1, 2, and 6 have been addressed in this release, with the rest being either invalid or intentionally left. Note that some of these bullets overlap.
{% endhint %}

* `ConstantTime.IsAllZeros()` now throws `ArgumentOutOfRangeException` if the buffer is empty. I'm arguing that this isn't a breaking change because the method name and documentation implies that the buffer has a length. I don't think there's any legitimate use case for allowing an empty buffer, so I'm not sure why this was done before.
* `Spans.Concat()` methods now throw `ArgumentException` if the buffer overlaps with any of the inputs because this is a misuse of these methods that can cause corruption. Tests have been updated for this. Again, this feels more like a bug fix than a breaking change, with nobody using these methods properly being affected.
* `Argon2id.VerifyHash()` and `NeedsRehash()` now check that everything after the first `0x00` byte in the password hash string is also zero. Test vectors have been added to cover this. libsodium expects null-terminated strings, and Geralt internally does this if the user provides a non-terminated string.
* `IncrementalBLAKE2b` now stores the hash size when caching to avoid someone reinitialising and restoring a cached state with a different hash size. `InvalidOperationException` is thrown if someone tries to do this, which can already be thrown in this method.
* `Incremental` and `GuardedHeapAllocation` class finalizers can no longer throw an exception when multiple threads are being used. Finalizers aren't meant to throw.
* The internal `Sodium.Initialize()` function now uses `lock` to ensure that initialisation always occurs only once regardless of multiple threads. This was the original patch in v4.2.0 before I switched to `Interlocked` for consistency with the `IDisposable` classes without thinking, which introduced an early return scenario.

## v4.2.0 - 2026-05-24

#### Changed

* Updated the test packages.

#### Fixed/Security

{% hint style="success" %}
Thank you to Alexey Avramov ([@hakavlad](https://github.com/hakavlad)) for providing AI code audits of Geralt [v4.1.0](https://github.com/samuel-lucas6/Geralt/releases/tag/v4.1.0) over email, which identified several Informational-Medium issues. Some were new from rerunning the models, and others were repeats or due to failed patching because I rushed parts of the last release.

Claude 4.7 Opus bullets 1, 4, and 6; GPT-5.5 Pro bullets 2, 3, 5, 6, 9, and 11; and Gemini 3.1 Pro bullets 2 and 6 have been addressed in this release, with the rest being either invalid or intentionally left. Note that some of these bullets overlap.
{% endhint %}

* `Incremental` and `GuardedHeapAllocation` class methods now use `Interlocked` correctly for thread safety and throw `InvalidOperationException` when accessed from multiple threads simultaneously. Nobody should be trying to use `IDisposable` classes from multiple threads (most .NET `IDisposable` classes are not thread-safe), but this provides protection for people who do. [`lock` statements](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/lock) can be used in calling code to avoid these exceptions when using multiple threads.
* `Interlocked` is also now used properly in the internal function `Sodium.Initialize()`, although libsodium initialisation should be thread-safe anyway, so this is more of a precaution.
* `Incremental` class states are now allocated after validation instead of on creation. It was done this way before to match the previous `struct` approach.
* Enum parameters are now validated for `IncrementalXChaCha20Poly1305.EncryptChunk()` and `Encodings.GetToBase64BufferSize()`/`ToBase64()`/`GetFromBase64BufferSize()`/`FromBase64()`, throwing `ArgumentOutOfRangeException` if the user passes a value that's not defined within the enum. No other classes use enums. Tests were added to cover these scenarios. This was overlooked.
* A `try-finally` pattern is now used for all internal zeroing. `X25519.DeriveSharedKey()` could theoretically have thrown without internal buffers being cleared, but internal uses of functions shouldn't throw due to validation/correct usage.
* `Sodium.Initialize()` was added to `Encodings.GetToBase64BufferSize()`, although it probably isn't required here. Every other function was correctly initialising libsodium before calling a libsodium function.
* `GuardedHeapAllocation.Dispose()` now has a guard clause before freeing the pointer for consistency with the other `IDisposable` classes.

## v4.1.0 - 2026-05-04

#### Changed

* Updated to .NET MAUI 10 because .NET 9 is shortly going [out of support](https://dotnet.microsoft.com/en-us/platform/support/policy/maui).
* Updated libsodium to [v1.0.22](https://github.com/jedisct1/libsodium/releases/tag/1.0.22-RELEASE). However, new features like post-quantum crypto and XOFs are being saved for a future release.
* Updated the test packages.

#### Fixed/Security

{% hint style="success" %}
Thank you to Alexey Avramov ([@hakavlad](https://github.com/hakavlad)) for providing AI code audits of Geralt v4.0.1 in [#21](https://github.com/samuel-lucas6/Geralt/discussions/21), which identified several Informational-Medium issues.

Claude 4.7 Opus bullets 1, 2, 3, and 8; GPT-5.4 Pro bullets 1 and 2; and Gemini 3.1 Pro bullets 1, 2, and 3 have been addressed in this release, with the rest being either invalid or intentionally left.

This was unfortunately shared at a time when I've been busy IRL, so I'm sorry for the delay. I'm still in the process of reviewing/updating the entire documentation but continue to be busy in May.
{% endhint %}

* The `Incremental` classes now use `Interlocked` for better thread safety.
* Libsodium initialisation now uses `Interlocked` in two places instead of one for better thread safety.
* Unmanaged memory is now used for `Incremental` states to allow secure erasure correctly and state alignment (in some cases) for performance.
* The finalisation was moved in `IncrementalEd25519ph.FinalizeAndVerify()` until after the return value, although this function shouldn't error.
* `Encodings.GetFromHexBufferSize()` and `GetFromBase64BufferSize()` now throw `ArgumentException` for non-ASCII characters in `hex`/`base64`. Some other validation code has also been improved for these methods, and test vectors have been added to cover Unicode characters.
* `Argon2id.VerifyHash()` and `NeedsRehash()` now throw `FormatException` if the password hash string contains non-ASCII characters or doesn't end in or allow space for a null terminator. Unicode and null-terminated test vectors have been added to cover these scenarios.
* An issue with an empty byte array `personalization` (rather than an empty span) was fixed in `HChaCha20.DeriveKey()`, with a test change for coverage. Other places in the code where this could happen (optional parameters where libsodium doesn't take a length) were checked and found to be fine.
* The interop return value for `sodium_base64_encoded_len()` has been corrected, with a `checked` statement added to check for overflows in `Encodings.GetToBase64BufferSize()`. This means the function can now throw `OverflowException`, but this should never realistically be reached.

## v4.0.1 - 2026-03-22

{% hint style="warning" %}
If you're updating from v3, please see the [v4.0.0](https://github.com/samuel-lucas6/Geralt/releases/tag/v4.0.0) release notes for **breaking changes** and other improvements. Updates to the [documentation](https://www.geralt.xyz/) are still in progress.
{% endhint %}

#### Fixed

* The .NET MAUI targeting in [v4.0.0](https://github.com/samuel-lucas6/Geralt/releases/tag/v4.0.0) (hopefully), which was broken by a `.csproj` file mistake affecting the GitHub Actions workflow artifact. In future, I should extract the package to check the `lib` folder before making a release...

## v4.0.0 - 2026-03-22

{% hint style="warning" %}
This release contains **breaking changes** (listed as `[BREAKING]:`) to improve the API as a result of a full API review. Please update your code accordingly. Updates to the [documentation](https://www.geralt.xyz/) will occur after this release but may take some time.
{% endhint %}

{% hint style="success" %}
The documentation is now being archived [here](https://github.com/samuel-lucas6/geralt-docs) if you want an old or offline copy. A release will be made for each version of Geralt.
{% endhint %}

#### Added

* \[BREAKING]: An optional BLAKE2b `personalization` parameter for `X25519.DeriveSenderSharedKey()` and `DeriveRecipientSharedKey()`. This moves the position of the `preSharedKey` parameter because personalization is more important/likely to be used.
* An `X25519.PersonalizationSize` constant for use with `DeriveSenderSharedKey()` and `DeriveRecipientSharedKey()`.
* `X25519.DeriveSenderSharedKey()` and `DeriveRecipientSharedKey()` now support different shared key output sizes, allowing compatibility with libsodium's `crypto_kx_client_session_keys()`/`crypto_kx_server_session_keys()`.
* Support for a personalization/salt in `BLAKE2b.ComputeHash()`. This is like `BLAKE2b.DeriveKey()` but unkeyed, with an empty personalization allowed if there's a salt.
* Support for an optional personalization/salt in `IncrementalBLAKE2b`. This is like `BLAKE2b.DeriveKey()` but incremental, and you can do personalized/salted hashing without a key too.
* `IncrementalBLAKE2b.SaltSize` and `PersonalizationSize` constants for incremental key derivation.
* An `Ed25519.GetSeed()` function to go alongside the public key function, which is equivalent to slicing the private key.
* Support for generics in `Validation`. The methods dealing with lengths are restricted to `IBinaryInteger<T>`, and the methods looking at values are either `T?`, `IEnumerable<T?>`, or `IEnumerable<T[]?>`.
* `Validation.GreaterThan()`, `LessThan()`, `Between()` (exclusive), `HasNoNullValues()`, and `HasNoNullOrEmptyValues()` methods. The last two also support jagged arrays.
* `Encodings.GetToHexBufferSize()`, `GetFromHexBufferSize()`, `GetToBase64BufferSize()`, and `GetFromBase64BufferSize()` functions for calculating the output buffer size when doing hex/Base64 encoding and decoding. This also adds extra validation about hex/Base64 lengths and `ignoreChars` characters, throwing `ArgumentOutOfRangeException`, `ArgumentException`, or `OverflowException` (in some cases).
* `SecureRandom.GeneratePassphrase()` now supports custom wordlists. This should probably just be used for English wordlists. Wordlists must only contain printable characters, no empty lines, and no spaces. Otherwise, `FormatException` will be thrown. See `GetWordlist()` for an example of how to format the wordlist.
* `SecureRandom.GetPassphraseBufferSize()` to figure out the maximum length of the output buffer for `GeneratePassphrase()`. After passphrase generation, the buffer needs to be sliced using `out int passphraseSize` to remove the unused space.
* `SecureRandom.GetWordlist()` to retrieve the bundled [EFF Long Wordlist](https://www.eff.org/dice) minus hyphenated words, leaving 7772 words.
* `SecureRandom.MinCharacterSetSize`, `MinLongestWordSize`, `MaxLongestWordSize`, and `MinWordlistSize` constants used in validation.
* A `SecureRandom.LongestWordSize` enum for the longest word in popular passphrase wordlists, which can be used when creating a buffer with `GetPassphraseBufferSize()`.
* `X25519.MinSharedKeySize` and `MaxSharedKeySize` constants for validation purposes with `DeriveSenderSharedKey()`/`DeriveRecipientSharedKey()`.
* An `Argon2id.HashSize` constant to replace `MaxHashSize`, which caused confusion.
* `Encodings.HexCharacterSet`, `Base64CharacterSet`, `Base64UrlCharacterSet`, and `Base64FullCharacterSet` constants.
* `BLAKE2b.BlockSize` and `IncrementalBLAKE2b.BlockSize` constants for alignment purposes.
* `Poly1305.BlockSize` and `IncrementalPoly1305.BlockSize` constants for alignment purposes.
* New tests and test vectors for better coverage, including a dedicated test class for the `Validation` methods.
* .NET 10 as a target for the tests.
* GitHub Actions tests now also cover `windows-arm64`, `linux-arm64`, and `macos-x64` (Intel).
* Android as an MAUI target because Android binaries were [added](https://github.com/jedisct1/libsodium/releases/tag/1.0.21-RELEASE) to the libsodium NuGet package. However, this hasn't been tested.
* A `.gitignore` file.

#### Changed

* \[BREAKING]: `Validation.EqualToSize()` has been renamed to `EqualTo()`, `SizeBetween()` has become `BetweenOrEqualTo()`, `NotLessThanMin()` has become `GreaterThanOrEqualTo()`, `MultipleOfSize()` has become `MultipleOf()`, `NotGreaterThanMax()` has become `LessThanOrEqualTo()`, and `GreaterThanZero()` has been replaced by `GreaterThan()`.
* \[BREAKING]: `Encodings.ToHex()`, `FromHex()`, `ToBase64()`, and `FromBase64()` now use span buffers for the output and string inputs.
* \[BREAKING]: `Argon2id.ComputeHash()`, `VerifyHash()`, and `NeedsRehash()` now use a span char buffer for the password hash. This improves the experience if you want to use strings whilst avoiding copies in memory if you don't.
* \[BREAKING]: `SecureRandom.GetPassphrase()` has been renamed to `GeneratePassphrase()` and now uses span buffers for the output and an optional custom wordlist (see Added section). This tries to eliminate copies of the passphrase in memory. Importantly, you must use `out int passphraseSize` to slice the output buffer to the correct length.
* \[BREAKING]: `SecureRandom.GetString()` has been renamed to `GenerateString()` and now uses span buffers for the output and character set. This tries to eliminate copies of the random string in memory.
* \[BREAKING]: Within `IncrementalXChaCha20Poly1305`, the `decryption` bool parameter has been renamed to `encryption`, which reverses the meaning of `true`/`false`. Furthermore, the parameter has been moved so the buffers come first.
* \[BREAKING]: Within `IncrementalXChaCha20Poly1305`, `Push()` has been renamed to `EncryptChunk()`, and `Pull()` has been renamed to `DecryptChunk()`.
* \[BREAKING]: The `Padding` class has been renamed to `Iso78164Padding` in case other padding algorithms are added in the future.
* \[BREAKING]: `Padding.GetPaddedLength()` has become `Iso78164Padding.GetPaddedBufferSize()`.
* \[BREAKING]: `Iso78164Padding.GetPaddedBufferSize()` now takes a span buffer instead of an integer for consistency with other similar functions (e.g., they have validation based on the content).
* \[BREAKING]: `Ed25519.ComputePublicKey()` has been renamed to `GetPublicKey()` since it just copies data from the private key.
* \[BREAKING]: `Ed25519.GetX25519PublicKey()` and `GetX25519PrivateKey()` have been renamed to `ComputeX25519PublicKey()` and `ComputeX25519PrivateKey()` respectively.
* \[BREAKING]: `Argon2id.MinHashSize` and `MaxHashSize` have been made private for internal validation purposes only.
* \[BREAKING]: The `BLAKE2b.PersonalSize` constant has been renamed to `PersonalizationSize`.
* \[BREAKING]: The `HChaCha20.PersonalSize` constant has been renamed to `PersonalizationSize`.
* \[BREAKING]: `SecureRandom.MinStringLength` and `MaxStringLength` have been renamed to `MinStringSize` and `MaxStringSize` respectively.
* \[BREAKING]: `SecureRandom.SymbolChars` now matches [OWASP's Password Special Characters](https://owasp.org/www-community/password-special-characters) list minus the space character.
* \[BREAKING]: The `Encodings.HexIgnoreChars` and `Base64IgnoreChars` constants have been updated to include more characters and are no longer used by default.
* \[BREAKING]: The `ChaCha20`/`XChaCha20` `Encrypt()` and `Decrypt()` functions now throw `OverflowException` when a counter overflow is detected instead of `CryptographicException`.
* \[BREAKING]: `GuardedHeapAllocation` now throws `InsufficientMemoryException` instead of `OutOfMemoryException`.
* \[BREAKING]: `Iso78164Padding.GetPaddedBufferSize()` now uses a `checked` statement, which may throw `OverflowException`.
* \[BREAKING]: `Iso78164Padding.Pad()` now throws `CryptographicException` on a libsodium error instead of `ArgumentOutOfRangeException`.
* \[BREAKING]: `HChaCha20.DeriveKey()` now checks that the `personalization` has [asymmetry](https://link.springer.com/article/10.1007/s00145-018-9297-9), throwing `ArgumentException` if not.
* \[BREAKING]: `Encodings.ToHex()` and `ToBase64()` now throw `CryptographicException` if there's a libsodium error instead of `FormatException`.
* \[BREAKING]: `SecureRandom.GeneratePassphrase()` now requires `separatorChar` to be a printable character, throwing `ArgumentException` if not.
* \[BREAKING]: `SecureRandom.GenerateString()` now requires a character set size of `SecureRandom.MinCharacterSetSize`, corresponding to `SecureRandom.MinUpperBound`.
* `Argon2id.MinHashSize` has been reduced to allow non-libsodium hashes to be verified.
* `CryptographicOperations.ZeroMemory()` has been replaced everywhere internally with `SecureMemory.ZeroMemory()` to use libsodium.
* `Incremental` class states are now pinned and zeroed using libsodium, with class finalizers added for unpinning. This is in case the `Finalize()` method isn't reached, which internally zeros the state.
* `SecureMemory.ZeroMemory()` now uses `MethodImplOptions.NoInlining` and `MethodImplOptions.NoOptimization` as a precaution, like .NET's `CryptographicOperations.ZeroMemory()`.
* Within `Encodings`, `stackalloc` is now used for small allocations instead of `GC.AllocateArray<>()` for performance.
* Error messages and parameter names have been improved/made more consistent.
* Code improvements, like the elimination of duplicate code, parameter names being specified when passing an empty value or discarding a value, additional zeroing of temporary buffers, new interop constants, etc.
* Tests have been improved (e.g., `Dictionary` is used for `_Tampered` tests, `Assert.ThrowsExactly<>()` is used, variable names are more consistent, incremental constants are used in `Incremental` test classes, etc).
* Classes/tests that aren't cryptographic algorithms have been moved to a `Helpers` folder. However, the namespace is the same.
* `Incremental` class tests have been moved to separate test classes.
* `Incremental` interop has been moved to separate files.
* The interop functions/constants now specify the cryptographic algorithm name when available (e.g., `crypto_pwhash_argon2id_SALTBYTES` instead of `crypto_pwhash_SALTBYTES`).
* Only libsodium [point releases](https://doc.libsodium.org/doc/quickstart#what-is-the-difference-between-point-releases-and-stable-releases) (not stable releases) are now checked when validating the version of the libsodium binary.
* Switched the project file from `.sln` to `.slnx`.
* Updated to [.NET MAUI 9](https://dotnet.microsoft.com/en-us/platform/support/policy/maui), which will be replaced with .NET MAUI 10 in the next release.
* Updated the version of libsodium to [1.0.21](https://github.com/jedisct1/libsodium/releases/tag/1.0.21-RELEASE).
* Updated the test packages.
* Updated the README and SECURITY policy.

#### Removed

* \[BREAKING]: `SecureMemory.LockMemory()`, `UnlockAndZeroMemory()`, and `SecureMemory.PageSize` because memory locking is complicated/error prone due to memory moving and page boundary alignment requirements. Furthermore, the `GuardedHeapAllocation` class does memory locking for you.
* \[BREAKING]: `BLAKE2b.StreamBufferSize` and `BLAKE2b.ComputeHash()` with a `Stream` because it was inconsistent with the rest of the API. See [here](https://github.com/samuel-lucas6/Geralt/blob/41972af7b385dcab6e7295b1a25545b7ceebd575/src/Geralt/Crypto/BLAKE2b.cs#L29) for how to implement this yourself using `IncrementalBLAKE2b`.

#### Fixed

* \[BREAKING]: The validation in the `Spans` class has been condensed, with a single `checked` statement per method, which may throw `OverflowException`.
* `SecureMemory.ZeroMemory()` no longer throws `ArgumentOutOfRangeException` when the buffer is empty.
* `ObjectDisposedException` is no longer thrown when `Dispose()` is called twice for the `Incremental` classes (nothing is thrown).
* Finalization state updates for `Incremental` classes now happen after the libsodium return value has been checked in all cases. However, this should have been the case before when it mattered (aka libsodium never returns 0/error for some functions).
* The `GuardedHeapAllocation` class now has a finalizer in case the user forgets to call `Dispose()` because there's a pointer (an unmanaged resource).
* Error messages for buffer length validation now include `".Length"` (equivalent to `fullnameof(buffer.Length)`).
* Patch/[stable releases](https://doc.libsodium.org/doc/quickstart#what-is-the-difference-between-point-releases-and-stable-releases) of the [libsodium NuGet package](https://www.nuget.org/packages/libsodium/) are now allowed rather than having just one pinned version. However, point releases still require Geralt to be updated.

## v3.3.0 - 2024-12-15

#### Added

* A `SecureMemory` class that contains methods for zeroing byte arrays, char arrays, and strings as well as locking/unlocking byte arrays. Note that arrays must be [pinned](https://www.geralt.xyz/secure-memory) for these methods to work properly.
* Guarded heap allocations, which can be used instead of regular allocations for additional security. However, there's a performance penalty, and this functionality shouldn't be used for large amounts of variables/data due to system limits. Make sure you read the new [documentation](https://www.geralt.xyz/secure-memory).
* A `Validation.MultipleOfSize()` method to check that an integer is a multiple of another integer.

#### Changed

* Every method in all the incremental classes now checks whether the object has been disposed, throwing `ObjectDisposedException` if so.
* The test packages have been updated.

#### Fixed

* A bug with `SecureRandom.GetPassphrase()` returning passphrases containing `'\r'` characters on Windows. This wasn't a problem in .NET 6 and wasn't detectable in a library context, only in a CLI application.
* The libsodium package has been updated, which fixes `win-arm64` support [not working](https://github.com/jedisct1/libsodium/issues/1430).

## v3.2.0 - 2024-11-17

#### Added

* The `IncrementalBLAKE2b` state can now be [cached](https://www.geralt.xyz/message-authentication#incrementalblake2b). This means static data at the beginning (e.g., the same key) can be processed just once for repeated calls, improving performance. Note that `CacheState()` can only cache the state once. Each subsequent call will overwrite the previously cached state. This was done for backwards compatibility, to allow zeroing the cached state, and because you shouldn't be able to cache the state with `IncrementalPoly1305`, for example.

#### Changed

* Now using some new language features, like [LibraryImport](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation).
* The state in incremental classes now gets zeroed if there's an exception/`Dispose()` is manually called.
* The byte array buffers in `Encodings.ToHex()` and `Encodings.ToBase64()` now get zeroed after a string is returned.
* `BLAKE2b.ComputeHash()` with a stream now throws `InvalidOperationException` if the stream can't be read. Previously, this would've been a `NotSupportedException` from the .NET API.
* Tests are now run on .NET 9 as well as .NET 8.
* The test packages have been updated.

#### Fixed

* `stackalloc` is no longer used for `Encodings.ToHex()` and `Encodings.ToBase64()`, which [could've](https://vcsjones.dev/stackalloc/) resulted in a `StackOverflowException` with large enough inputs.

#### Removed

* .NET 6 is no longer targeted since it's [out of support](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core).

## v3.1.0 - 2024-09-01

#### Added

* A `Reinitialize()` function for the `Incremental` classes. This saves you having to create another `using` statement in some scenarios.

#### Changed

* Added support for iOS, tvOS, and Mac Catalyst. **Help is wanted to test this** ([#7](https://github.com/samuel-lucas6/Geralt/issues/7)).
* No longer targeting .NET 7 as it's [out of support](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core).
* Updated the libsodium version to 1.0.20.
* Minor test improvements.

#### Deprecated

* .NET 6 support will be dropped in [November 2024](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core), which will allow some newer language features to be used.

## v3.0.1 - 2023-11-19

#### Changed

* Now targeting .NET 8.
* Now testing on .NET 8.
* Minor test improvements.

## v3.0.0 - 2023-10-01

{% hint style="warning" %}
This release contains **breaking changes** to public constants, function/class renaming, and more validation to improve the API. Please update your code accordingly. Updates to the [documentation](https://www.geralt.xyz/) are in progress.
{% endhint %}

#### Added

* AEGIS-128L and AEGIS-256, which are [fast](https://eprint.iacr.org/2023/523), [key committing](https://eprint.iacr.org/2022/268) AES-based AEAD schemes that were finalists in the [CAESAR competition](https://competitions.cr.yp.to/caesar-submissions.html). They are both preferable to AES-GCM and AES-OCB. The implementations also don't require hardware support to work, although performance will be affected and side-channels may exist.
* `FinalizeAndVerify()` methods for `IncrementalBLAKE2b` and `IncrementalPoly1305`. These are like the non-incremental `VerifyTag()` methods.
* Validation to prevent updating the state after finalizing or finalizing twice in Incremental classes. This includes after specifying `ChunkFlag.Final` in `IncrementalXChaCha20Poly1305`.
* Validation that encoded password hash strings are for Argon2id, not Argon2i/Argon2d.
* A check for counter overflows with `XChaCha20` to match `ChaCha20`.
* A `NotGreaterThanMax()` validation function.
* A link to the release notes on NuGet.
* GitHub Actions tests on `linux-musl-x64`.

#### Changed

* Updated to [libsodium v1.0.19](https://github.com/jedisct1/libsodium/releases/tag/1.0.19-RELEASE).
* The following constants have been changed: `BLAKE2b.HashSize`, `BLAKE2b.MinKeySize`, `IncrementalBLAKE2b.HashSize`, `IncrementalBLAKE2b.MinKeySize`, `X25519.MinPreSharedKeySize`, `Argon2id.MinKeySize`, and `Argon2id.MinMemorySize`. `Argon2id.HashPrefix` has also been made private.
* `IncrementalEd25519` has been renamed to `IncrementalEd25519ph`.
* `IncrementalEd25519ph.Verify()` has been renamed to `IncrementalEd25519ph.FinalizeAndVerify()`.
* Hyphenated words have been removed from the passphrase wordlist.
* Various exception messages have been rephrased.
* Code/test improvements.

## v2.1.0 - 2023-05-13

#### Added

* `IncrementalEd25519`, which uses Ed25519ph.
* Support for an empty salt with `BLAKE2b.DeriveKey()`. This is equivalent to a 128-bit all-zero salt. This makes sense when you only need to derive a single key or when there's no need for salting (e.g. ephemeral keys are involved in a key exchange).
* Preparations to support iOS in the future.

#### Changed

* American spellings (initialize, finalize, personalization, capitalize, etc) are now used for consistency.
* More thorough testing.
* Code improvements.

An upcoming release will likely change some constants (e.g. `BLAKE2b.HashSize`) to be consistent with libsodium. This will be a breaking change.

## v2.0.0 - 2022-11-30

#### Added

* `IncrementalXChaCha20Poly1305`, which is a wrapper around [crypto\_secretstream\_\*()](https://doc.libsodium.org/secret-key_cryptography/secretstream) for chunked stream/file encryption. You can read the Geralt documentation [here](https://www.geralt.xyz/authenticated-encryption/stream-and-file-encryption).
* Constants for `IncrementalBLAKE2b` and `IncrementalPoly1305`, which are identical to the `BLAKE2b` and `Poly1305` constants.
* Support for [.NET 7](https://dotnet.microsoft.com/en-us/download/dotnet/7.0).

#### Changed

* `DeriveSenderSharedSecret()` has been renamed to `DeriveSenderSharedKey()` for clarity.
* `DeriveRecipientSharedSecret()` has been renamed to `DeriveRecipientSharedKey()`.
* `ComputeXCoordinate()` has been renamed to `ComputeSharedSecret()`. The above functions should still be preferred to prevent accidental [vulnerabilities](https://datatracker.ietf.org/doc/html/rfc7748#section-7).
* The `Validation` class has been made public because it's useful for [custom constructions](https://github.com/samuel-lucas6/kcChaCha20-Poly1305) without having to have hideous if statements everywhere.

#### Removed

* The `BLAKE2bHashAlgorithm` class because it returned a byte array. It has been replaced with a `BLAKE2b.ComputeHash()` function that takes a `Stream` and `IncrementalBLAKE2b` for keyed hashing.

## v1.3.0 - 2022-09-01

#### Added

* `IncrementalBLAKE2b`.
* `IncrementalPoly1305`.

#### Changed

* `BLAKE2bHashAlgorithm` now uses `IncrementalBLAKE2b`.
* The validation has been reordered for ChaCha20-Poly1305 and XChaCha20-Poly1305.

## v1.2.0 - 2022-08-30

#### Added

* Non-XOR methods for ChaCha20 and XChaCha20, which can be used to implement (X)ChaCha20-Poly1305.

## v1.1.0 - 2022-08-29

#### Added

* The internal counter for ChaCha20 and XChaCha20 can now be accessed. Overflow checking is done for the ChaCha20 counter.

#### Changed

* `Spans.Concat()` now accepts empty spans like `Arrays.Concat()` did.
* The incremental BLAKE2b state handling now matches libsodium-core.

#### Removed

* The `Arrays` class because spans and `Spans.Concat()` should be used instead.

## v1.0.3 - 2022-08-22

#### Changed

* The initialisation error messages.

## v1.0.2 - 2022-08-12

#### Fixed

* Empty passwords, messages, and plaintexts/ciphertexts are now allowed.
* An exception message typo.

## v1.0.1 - 2022-08-09

#### Changed

* The [Sodium](https://github.com/samuel-lucas6/Geralt/blob/v1.0.1/src/Geralt/Interop/Sodium.cs) class for [libsodium initialisation](https://doc.libsodium.org/quickstart#boilerplate) is now internal because it doesn't need to be publicly accessible.

## v1.0.0 - 2022-08-09

N/A - initial release.
