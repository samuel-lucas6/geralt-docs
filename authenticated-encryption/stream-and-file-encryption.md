# Stream and file encryption

## Purpose

Encrypting streams and files should be done in chunks. This keeps memory usage low and detects corrupted chunks early. However, this is more complicated than one would expect because you need to prevent chunks from being removed, truncated, duplicated, and reordered.

This [high-level API](https://doc.libsodium.org/secret-key_cryptography/secretstream) uses [XChaCha20-Poly1305](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-xchacha#section-2) internally whilst handling these issues for you. Plaintext chunks can have different sizes, there is no practical stream length limit, associated data is supported, and rekeying is possible midway through encrypting the stream.

The optional associated data is useful for authenticating file headers, version numbers, timestamps, counters, and so on. It can be used to prevent [confused deputy attacks](https://cloud.google.com/kms/docs/additional-authenticated-data#confused_deputy_attack_example) and [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). It is not encrypted nor part of the ciphertext. It must be reproduceable or stored somewhere for decryption to be possible.

{% hint style="success" %}
For file encryption, optimal chunk sizes include 16 KiB, 32 KiB, and 64 KiB. Pick one and use it for everything but the last chunk, which will likely be smaller.
{% endhint %}

## Usage

### IncrementalXChaCha20Poly1305

Initializes stream encryption or decryption using a key and header. For encryption, the header is filled with a random nonce. It **MUST** be sent/stored before the sequence of ciphertext messages because it is required to decrypt the stream.

```csharp
using var secretstream = new IncrementalXChaCha20Poly1305(Span<byte> header, ReadOnlySpan<byte> key, bool encryption)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`header` has a length not equal to `HeaderSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error initializing stream encryption/decryption.

### ChunkFlag

Before encryption, a flag is attached to each chunk to provide information about that chunk:

* `ChunkFlag.Message`: an ordinary plaintext chunk.
* `ChunkFlag.Boundary`: the end of a set of messages but not the end of the stream.
* `ChunkFlag.Rekey`: derive a new key after this chunk.
* `ChunkFlag.Final`: the last plaintext chunk, marking the end of the stream.

For file encryption, `ChunkFlag.Message` and `ChunkFlag.Final` are the only flags required. For online communication, `ChunkFlag.Boundary` and `ChunkFlag.Rekey` may also be useful.

{% hint style="danger" %}
During decryption, you **MUST** check that the last chunk had the `ChunkFlag.Final` flag. Otherwise, you **MUST** throw a [CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception) to detect stream truncation.
{% endhint %}

### EncryptChunk

Fills a span with the ciphertext chunk computed from a plaintext chunk and optional associated data. `ChunkFlag.Final` **MUST** be specified for the last plaintext chunk.

```csharp
secretstream.EncryptChunk(Span<byte> ciphertextChunk, ReadOnlySpan<byte> plaintextChunk, ReadOnlySpan<byte> associatedData, ChunkFlag chunkFlag = ChunkFlag.Message)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertextChunk` has a length not equal to `plaintextChunk.Length + TagSize`.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot encrypt on a decryption stream or after the final chunk without reinitializing.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error encrypting plaintext chunk.

### DecryptChunk

Verifies that the tag appended to the ciphertext chunk is correct for the given inputs. If verification fails, an exception is thrown. Otherwise, it fills a span with the decrypted ciphertext and returns the decrypted flag for that chunk.

To detect stream truncation, if you reach the end of the stream and `ChunkFlag.Final` was not returned, you **MUST** throw a [CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception).

```csharp
var chunkFlag = secretstream.DecryptChunk(Span<byte> plaintextChunk, ReadOnlySpan<byte> ciphertextChunk, ReadOnlySpan<byte> associatedData = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertextChunk` has a length less than `TagSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`plaintextChunk` has a length not equal to `ciphertextChunk.Length - TagSize`.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot decrypt on an encryption stream or after the final chunk without reinitializing.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid chunk authentication tag for the given inputs.

### Rekey

Explicitly rekeys without storing the `ChunkFlag.Rekey` flag. For decryption, this function **MUST** be called at the same stream location to avoid an exception when decrypting the following chunk.

```csharp
secretstream.Rekey()
```

#### Exceptions

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot rekey after the final chunk without reinitializing.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

### Reinitialize

Reinitializes stream encryption or decryption using a key and header. This avoids creating another using statement. For encryption, the header is filled with a random nonce. It **MUST** be sent/stored before the sequence of ciphertext messages because it is required to decrypt the stream.

```csharp
secretstream.Reinitialize(Span<byte> header, ReadOnlySpan<byte> key, bool encryption)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`header` has a length not equal to `HeaderSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`key` has a length not equal to `KeySize`.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error initializing stream encryption/decryption.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int KeySize = 32;
public const int HeaderSize = 24;
public const int TagSize = 17;
```

## Notes

{% hint style="danger" %}
If you intend to feed **multiple variable-length** inputs into the associated data, beware of [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Please read the [Concat](../advanced/concat.md) page for more information.
{% endhint %}

{% hint style="danger" %}
The key **MUST** be uniformly random. It can either be [randomly generated](../random-data.md#fill) or the output of a [KDF](../key-derivation.md). Furthermore, it **SHOULD** be rotated periodically (e.g., a different key per file).
{% endhint %}

{% hint style="warning" %}
As a _general_ rule, avoid compression before encryption. It can [leak information](https://iacr.org/archive/fse2002/23650264/23650264.pdf) and has been the cause of [several attacks](https://crypto.stackexchange.com/a/38055).
{% endhint %}
