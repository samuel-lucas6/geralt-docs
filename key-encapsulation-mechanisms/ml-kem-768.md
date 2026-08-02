# ML-KEM-768

## Purpose

When communicating with another party, you often need a way to establish a shared secret (symmetric) key without having an existing secure channel. An algorithm that allows this over a public (insecure) channel is called a key-establishment scheme.

Key encapsulation mechanisms (KEMs) are one type of key-establishment scheme, and they're the interface of choice for **post-quantum algorithms**, which protect against cryptographically relevant quantum computers (CRQCs). Even though CRQCs don't currently exist, an adversary can capture encrypted data that relies on [traditional key exchange](../key-exchange.md) algorithms ready for decryption when such a CRQC becomes available (a [store now, decrypt later attack](https://en.wikipedia.org/wiki/Harvest_now%2C_decrypt_later)).

Unlike a traditional key exchange, the sender's key pair isn't involved, a ciphertext needs to be sent to the recipient, and the shared secret is uniformly random. From the sending side, the algorithm is randomised rather than deterministic.

Here is how it works when doing one trip of communication (real protocols often do multiple trips):

1. **Key generation**: the recipient generates a key pair and shares the public key with the sender.
2. **Encapsulation**: the sender uses the recipient's public key to generate a shared secret and an associated ciphertext. This ciphertext is sent to the recipient.
3. **Decapsulation**: the recipient uses the ciphertext and their private key to compute the same shared secret.

[ML-KEM-768](https://csrc.nist.gov/pubs/fips/203/final) is the middle (192-bit) security strength variant of ML-KEM, which is one of the algorithms standardised by [NIST](https://csrc.nist.gov/Projects/post-quantum-cryptography/post-quantum-cryptography-standardization/selected-algorithms) as part of the Post-Quantum Cryptography (PQC) Standardization (competition) process. This variant provides protection against cryptanalysis advancements compared to ML-KEM-512 whilst being [lighter](https://pqshield.github.io/nist-sigs-zoo/kems/?s=ML-KEM) (smaller parameters/marginally faster) than ML-KEM-1024.

{% hint style="danger" %}
Private keys **MUST** **NOT** be shared. They **MUST** remain secret.
{% endhint %}

{% hint style="success" %}
Consider using [X-Wing](x-wing.md) instead, which uses X25519 + ML-KEM-768 to provide [classical security still](https://soatok.blog/2022/01/27/the-controversy-surrounding-hybrid-cryptography/) if ML-KEM-768 is unexpectedly broken. Whilst this is [arguably unnecessary](https://soatok.blog/2026/04/13/hybrid-constructions-the-post-quantum-safety-blanket/) and has [some drawbacks](https://www.ncsc.gov.uk/paper/next-steps-in-preparing-for-post-quantum-cryptography#section_5), it is a [popular/recommended approach](https://crypto.stackexchange.com/q/119797).
{% endhint %}

## Usage

### GenerateKeyPair

Fills a span with a randomly generated private key and another span with the associated public key.

```csharp
MLKEM768.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error generating key pair.

### GenerateKeyPair

Fills a span with a private key generated using a [random](../random-data.md#fill) seed and another span with the associated public key.

```csharp
MLKEM768.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey, ReadOnlySpan<byte> seed);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`publicKey` has a length not equal to `PublicKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`privateKey` has a length not equal to `PrivateKeySize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`seed` has a length not equal to `SeedSize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error generating key pair from seed.

### Encapsulate

Fills a span with the computed shared secret and another span with the ciphertext to send to the recipient based on the recipient's public key.

```csharp
MLKEM768.Encapsulate(Span<byte> sharedSecret, Span<byte> ciphertext, ReadOnlySpan<byte> recipientPublicKey);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedSecret` has a length not equal to `SharedSecretSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `CiphertextSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPublicKey` has a length not equal to `PublicKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid public key.

### Decapsulate

Fills a span with the computed shared secret based on the recipient's private key and the ciphertext the recipient was sent.

```csharp
MLKEM768.Decapsulate(Span<byte> sharedSecret, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> recipientPrivateKey);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedSecret` has a length not equal to `SharedSecretSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `CiphertextSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPrivateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Error performing decapsulation.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int PublicKeySize = 1184;
public const int PrivateKeySize = 2400;
public const int SeedSize = 64;
public const int SharedSecretSize = 32;
public const int CiphertextSize = 1088;
```

## Notes

{% hint style="warning" %}
Not all uses of traditional key exchange can be replaced in a straightforward manner by KEMs. For example, KEMs don't provide non-interactive key exchange (NIKE) functionality.
{% endhint %}

{% hint style="warning" %}
[ML-KEM](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon) public keys and ciphertexts are distinguishable from random (e.g., someone can tell that cryptography is being used). Therefore, it's not suitable for scenarios like plausible deniability and censorship-resistance without using a scheme such as [Kemeleon](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon), which is not implemented in libsodium.
{% endhint %}

{% hint style="info" %}
If you [read](https://csrc.nist.gov/pubs/sp/800/227/final) about post-quantum algorithms, you may see the terms 'encapsulation key' and 'decapsulation key'. These mean 'public key' and 'private key' but are specific to KEMs.
{% endhint %}
