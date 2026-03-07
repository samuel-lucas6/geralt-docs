# Digital signatures

## Purpose

A digital signature verifies the authenticity of a message and provides non-repudiation. This means any change to the message causes signature verification to fail, you know who signed the message, and someone cannot deny having signed a message.

Signing is done using a private key. The associated public key can then be publicly shared to allow others to verify signatures.

{% hint style="danger" %}
Private keys **MUST** **NOT** be shared. They **MUST** remain secret.
{% endhint %}

{% hint style="warning" %}
Generally, **avoid** using signatures with encryption and instead rely on **authenticated** [key exchange](key-exchange.md). You can find out more [here](https://neilmadden.blog/2018/11/14/public-key-authenticated-encryption-and-why-you-want-it-part-i/).
{% endhint %}

## Usage

### GenerateKeyPair

Fills a span with a randomly generated private key and another span with the associated public key.

```csharp
Ed25519.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Unable to generate key pair.

### GenerateKeyPair

Fills a span with a private key generated using a [random](random-data.md#fill) seed and another span with the associated public key.

```csharp
Ed25519.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey, ReadOnlySpan<byte> seed)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`seed` has a length not equal to `SeedSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Unable to generate key pair from seed.

### ComputePublicKey

Fills a span with the public key computed from a private key.

```csharp
Ed25519.ComputePublicKey(Span<byte> publicKey, ReadOnlySpan<byte> privateKey)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Unable to compute public key from private key.

### Sign

Fills a span with the signature for a message signed using a private key.

```csharp
Ed25519.Sign(Span<byte> signature, ReadOnlySpan<byte> message, ReadOnlySpan<byte> privateKey)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`signature` has a length not equal to `SignatureSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Unable to compute signature.

### Verify

Determines if a signature is valid for a message and public key. It returns `true` if the signature is valid and `false` otherwise.

```csharp
Ed25519.Verify(ReadOnlySpan<byte> signature, ReadOnlySpan<byte> message, ReadOnlySpan<byte> publicKey)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`signature` has a length not equal to `SignatureSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

### IncrementalEd25519ph

Provides support for computing/verifying a signature from a sequence of messages using [Ed25519ph](https://www.rfc-editor.org/rfc/rfc8032).

`IncrementalEd25519ph.Finalize()` fills a span with the signature for a chunked message signed using a private key.

`IncrementalEd25519ph.FinalizeAndVerify()` determines if a signature is valid for a chunked message and public key. It returns `true` if the signature is valid and `false` otherwise.

```csharp
using var ed25519ph = new IncrementalEd25519ph();
ed25519ph.Update(ReadOnlySpan<byte> message1);
ed25519ph.Update(ReadOnlySpan<byte> message2);
// Sign
ed25519ph.Finalize(Span<byte> signature, ReadOnlySpan<byte> privateKey);
// Or verify
bool valid = ed25519ph.FinalizeAndVerify(ReadOnlySpan<byte> signature, ReadOnlySpan<byte> publicKey)

// Avoid another using statement
ed25519ph.Reinitialize();
ed25519ph.Update(ReadOnlySpan<byte> message3);
ed25519ph.Finalize(Span<byte> signature, ReadOnlySpan<byte> privateKey);
```

{% hint style="warning" %}
This should **only** be used when the message is too large to fit into memory because prehashing is [theoretically weaker](https://cryptologie.net/article/497/eddsa-ed25519-ed25519-ietf-ed25519ph-ed25519ctx-hasheddsa-pureeddsa-wtf/) than regular signing.
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`signature` has a length not equal to `SignatureSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

The signature could not be computed.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Cannot update after finalizing or finalize twice (without reinitializing).

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int PublicKeySize = 32;
public const int PrivateKeySize = 64;
public const int SignatureSize = 64;
public const int SeedSize = 32;
```

## Notes

{% hint style="info" %}
If you want to use [BLAKE2b](hashing.md#incrementalblake2b) for prehashing instead of Ed25519ph, which uses SHA-512 internally, you can hash a domain separation constant (e.g. the protocol name) concatenated with the message and sign the 512-bit hash.
{% endhint %}

{% hint style="danger" %}
If you want to support prehashing as well as non-prehashed Ed25519 like in [Kryptor](https://www.kryptor.co.uk/)/[Minisign](https://jedisct1.github.io/minisign/), you **MUST** sign some data indicating whether prehashing was used or not. Otherwise, it may be [possible](https://github.com/jedisct1/minisign/issues/104) to create a forgery.
{% endhint %}

{% hint style="warning" %}
Ed25519 is vulnerable to [fault attacks](https://eprint.iacr.org/2017/1014.pdf). Techniques like causing [voltage glitches](https://cybermashup.files.wordpress.com/2017/10/practical-fault-attack-against-eddsa_fdtc-2017.pdf) on a chip (e.g. on an Arduino) can be used to recover the secret key and create valid signatures.

This should generally not concern you as it's mostly relevant for embedded devices and requires physical or remote access to the device. Furthermore, most countermeasures are ineffective. Prehashing or [hedged signatures](https://soatok.blog/2020/05/03/hedged-signatures-with-libsodium-using-dhole/) can help but will not prevent all attacks.
{% endhint %}
