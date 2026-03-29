# Key exchange

## Purpose

It is generally difficult to share a symmetric key with another party because it needs to remain secret. Fortunately, a sender and recipient can exchange public keys, which do not need to be secret.

Both parties can then compute the same shared secret using their private key and the other party's public key. This shared secret can be turned into a symmetric key (e.g. for [encryption](authenticated-encryption/xchacha20-poly1305.md)).

For [post-quantum security](https://www.bsi.bund.de/SharedDocs/Downloads/EN/BSI/Publications/Brochure/quantum-safe-cryptography.html?nn=433196), a pre-shared key can also be shared between the two parties via a **secure** (e.g. **encrypted**) channel.

{% hint style="danger" %}
Private keys **MUST** **NOT** be shared. They **MUST** remain secret.
{% endhint %}

{% hint style="info" %}
You often want to include ephemeral (one-time) key pairs in a key exchange as well. See the [Notes](key-exchange.md#notes) for examples of how to do this.
{% endhint %}

## Usage

### GenerateKeyPair

Fills a span with a randomly generated private key and another span with the associated public key.

```csharp
X25519.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey)
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
X25519.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey, ReadOnlySpan<byte> seed)
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
X25519.ComputePublicKey(Span<byte> publicKey, ReadOnlySpan<byte> privateKey)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Unable to compute public key from private key.

### DeriveSenderSharedKey

Fills a span with the shared key **for the sender** using their private key, a recipient public key, and an optional pre-shared key.

```csharp
X25519.DeriveSenderSharedKey(Span<byte> sharedKey, ReadOnlySpan<byte> senderPrivateKey, ReadOnlySpan<byte> recipientPublicKey, ReadOnlySpan<byte> preSharedKey = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedKey` has a length less than `MinSharedKeySize` or greater than `MaxSharedKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`senderPrivateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPublicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

If specified, `preSharedKey` has a length less than `MinPreSharedKeySize` or greater than `MaxPreSharedKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid recipient public key or unable to compute hash.

### DeriveRecipientSharedKey

Fills a span with the shared key **for the recipient** using their private key, the sender's public key, and an optional pre-shared key.

```csharp
X25519.DeriveRecipientSharedKey(Span<byte> sharedKey, ReadOnlySpan<byte> recipientPrivateKey, ReadOnlySpan<byte> senderPublicKey, ReadOnlySpan<byte> preSharedKey = default)
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedKey` has a length less than `MinSharedKeySize` or greater than `MaxSharedKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPrivateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`senderPublicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

If specified, `preSharedKey` has a length less than `MinPreSharedKeySize` or greater than `MaxPreSharedKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid sender public key or unable to compute hash.

### ComputeSharedSecret

Fills a span with the **unhashed** X25519 shared secret for a given sender private key and recipient public key.

```csharp
X25519.ComputeSharedSecret(Span<byte> sharedSecret, ReadOnlySpan<byte> senderPrivateKey, ReadOnlySpan<byte> recipientPublicKey)
```

{% hint style="danger" %}
1. The number of possible keys is limited to the group size (\~2^252), which is smaller than the key space.
2. Many (public key, private key) pairs produce the same shared secret. This can lead to [vulnerabilities](https://datatracker.ietf.org/doc/html/rfc7748#section-7).

Therefore, it is important to pass the shared secret [concatenated](advanced/concat.md) with the sender's public key and recipient's public key through a [KDF](key-derivation.md).
{% endhint %}

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedSecret` has a length not equal to `SharedSecretSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`senderPrivateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPublicKey` has a length not equal to `PublicKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid public key.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int PublicKeySize = 32;
public const int PrivateKeySize = 32;
public const int SeedSize = 32;
public const int SharedSecretSize = 32;
public const int SharedKeySize = 32;
public const int PersonalizationSize = 16;
public const int PreSharedKeySize = 32;
public const int MinSharedKeySize = 16;
public const int MaxSharedKeySize = 64;
public const int MinPreSharedKeySize = 16;
public const int MaxPreSharedKeySize = 64;
```

## Notes

{% hint style="success" %}
You generally want to use key exchange [patterns](https://noiseprotocol.org/noise.html#handshake-patterns) from the [Noise Protocol Framework](https://noiseprotocol.org/noise.html) or a protocol like Signal's [X3DH](https://www.signal.org/docs/specifications/x3dh/). A _rough_ summary of some popular Noise patterns can be found below.
{% endhint %}

{% hint style="warning" %}
You should [derive](key-derivation.md) different keys for each direction (sending/receiving) if using a [counter](constant-time.md#increment) nonce for [encryption](authenticated-encryption/chacha20-poly1305.md) so you do not have to wait for an acknowledgment after every message. This can be done by deriving 512 bits of output keying material and using the first 256 bits for the sender and the latter 256 bits for the recipient.
{% endhint %}

{% hint style="warning" %}
X25519 public keys are distinguishable from random. This makes it very difficult to hide that cryptography is being used if that is a goal. [Elligator](https://elligator.org/) can be used to do this, but it is not available in libsodium.
{% endhint %}

{% hint style="danger" %}
Do **NOT** use the same seed for [X25519](key-exchange.md#generatekeypair-1) and [Ed25519](digital-signatures.md#generatekeypair-1) key generation. Instead, read the [Ed25519 to X25519](advanced/ed25519-to-x25519.md) page.
{% endhint %}

### Non-Interactive Patterns

These are one-way patterns, so no back and forth between the sender and recipient is required. They are appropriate for offline applications (e.g. a file encryption program).

#### Authenticated Key Exchange

The recipient knows who sent the message, and only the recipient can decrypt the message. This can be achieved using the [K pattern](https://noiseprotocol.org/noise.html#one-way-handshake-patterns). The security properties are discussed [here](https://noiseexplorer.com/patterns/K/).

1. The sender generates an ephemeral key pair.
2. The sender computes an ephemeral shared secret using their ephemeral private key and the recipient's public key. The ephemeral private key is then erased from memory.
3. The sender also computes a long-term shared secret using their long-term private key and the recipient's public key.
4. The sender concatenates the ephemeral shared secret and long-term shared secret to form the input keying material for a KDF. The output keying material is used as the key to encrypt a message using an AEAD.
5. The sender's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the recipient.
6. The recipient reads the ephemeral public key, computes the ephemeral shared secret using their private key and the sender's ephemeral public key, computes the long-term shared secret using their private key and the sender's long-term public key, derives the encryption key using the same KDF, and decrypts the ciphertext.

The [X pattern](https://noiseprotocol.org/noise.html#one-way-handshake-patterns) can also be used, which is identical except that the sender's long-term public key is sent with the message under encryption instead of already being known to the recipient. The security properties are discussed [here](https://noiseexplorer.com/patterns/X/).

#### Deniable Authenticated Key Exchange

The recipient knows who sent the message, but both parties can decrypt the message. This means either party could have encrypted the message.

{% hint style="warning" %}
Noise does not provide a pattern for this, and it is weaker than non-deniable authenticated key exchange because a compromise of either party's private key is problematic.

Furthermore, the same shared secret will always be derived for the same (sender, recipient) pair, so you need to be even more careful about not reusing a nonce with the AEAD.
{% endhint %}

1. The sender computes a shared secret using their long-term private key and the recipient's public key.
2. The sender uses the shared secret as the input keying material for a KDF. The output keying material is used as the key to encrypt a message using an AEAD.
3. The recipient computes the shared secret using their private key and the sender's public key, derives the encryption key using the same KDF, and decrypts the ciphertext.

#### Anonymous Key Exchange

The recipient does not know who sent the message, and only the recipient can decrypt the message. This can be accomplished using the [N pattern](https://noiseprotocol.org/noise.html#one-way-handshake-patterns). The security properties are discussed [here](https://noiseexplorer.com/patterns/N/).

{% hint style="warning" %}
You generally want an **authenticated** key exchange because you often want to be sure that a message came from a certain person. Otherwise, an attacker can replace a message with one of their choosing, and the recipient will be none the wiser.
{% endhint %}

1. The sender generates an ephemeral key pair.
2. The sender computes the shared secret using their ephemeral private key and the recipient's public key. The ephemeral private key is then erased from memory.
3. The sender uses the shared secret as input keying material to a KDF. The output keying material is used as the key to encrypt a message using an AEAD.
4. The sender's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the recipient.
5. The recipient reads the ephemeral public key, computes the shared secret using their private key and the sender's ephemeral public key, derives the encryption key using the same KDF, and decrypts the ciphertext.

### Interactive Patterns

These are two-way patterns, so back and forth between the sender and recipient is required. They are appropriate for online applications and can provide [better](https://noiseprotocol.org/noise.html#payload-security-properties) security properties than the one-way patterns.

#### Authenticated Key Exchange

The recipient knows who sent the message, and you [eventually](https://noiseexplorer.com/patterns/KK/) benefit from resistance to key compromise impersonation and strong forward secrecy. This can be achieved using the [KK pattern](https://noiseprotocol.org/noise.html#interactive-handshake-patterns-fundamental).

1. The sender generates an ephemeral key pair.
2. The sender computes an ephemeral shared secret using their ephemeral private key and the recipient's long-term public key.
3. The sender also computes a long-term shared secret using their long-term private key and the recipient's long-term public key.
4. The sender concatenates the ephemeral shared secret and long-term shared secret to form the input keying material for a KDF. The output keying material is used as the key to encrypt a message using an AEAD.
5. The sender's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the recipient.
6. The recipient reads the ephemeral public key, computes the ephemeral shared secret using their private key and the sender's ephemeral public key, computes the long-term shared secret using their private key and the sender's long-term public key, derives the encryption key using the same KDF, and decrypts the ciphertext.
7. The recipient generates an ephemeral key pair.
8. The recipient computes an ephemeral shared secret using their ephemeral private key and the sender's ephemeral public key.
9. The recipient computes another ephemeral shared secret using their ephemeral private key and the sender's long-term public key. The ephemeral private key is then erased from memory.
10. The recipient concatenates both ephemeral shared secrets to form the input keying material for a KDF. Importantly, the previous derived encryption key is included in this KDF usage (as a salt by the spec, but the info parameter could be used). The output keying material is used as the key to encrypt a message using an AEAD.
11. The recipient's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the sender.
12. The sender reads the ephemeral public key, computes the ephemeral shared secret using their ephemeral private key and the sender's ephemeral public key, erases their ephemeral private key, computes the other shared secret using their long-term private key and the recipient's ephemeral public key, and then derives the new encryption key in the same way the recipient did, which allows them to decrypt the ciphertext.
13. Subsequent messages can be encrypted using the same key and an incremented nonce.

#### Anonymous Key Exchange

The recipient does not know who sent the message. This can be accomplished using the [NK pattern](https://noiseprotocol.org/noise.html#interactive-handshake-patterns-fundamental). The security properties are discussed [here](https://noiseexplorer.com/patterns/NK/).

1. The sender generates an ephemeral key pair.
2. The sender computes the shared secret using their ephemeral private key and the recipient's public key.
3. The sender uses the shared secret as input keying material to a KDF. The output keying material is used as the key to encrypt a message using an AEAD.
4. The sender's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the recipient.
5. The recipient reads the ephemeral public key, computes the shared secret using their private key and the sender's ephemeral public key, derives the encryption key using the same KDF, and decrypts the ciphertext.
6. The recipient generates an ephemeral key pair.
7. The recipient computes an ephemeral shared secret using their ephemeral private key and the sender's ephemeral public key. The ephemeral private key is then erased from memory.
8. The recipient uses the ephemeral shared secret as input keying material to a KDF. Importantly, the previous derived encryption key is included in this KDF usage (as a salt by the spec, but the info parameter could be used). The output keying material is used as the key to encrypt a message using an AEAD.
9. The recipient's ephemeral public key is prepended to the ciphertext, and the ciphertext is sent to the sender.
10. The sender reads the ephemeral public key, computes the ephemeral shared secret using their ephemeral private key and the sender's ephemeral public key, erases their ephemeral private key, and then derives the new encryption key in the same way the recipient did, which allows them to decrypt the ciphertext.
11. Subsequent messages can be encrypted using the same key and an incremented nonce.
