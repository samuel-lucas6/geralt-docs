# X-Wing

## Purpose

When communicating with another party, you often need a way to establish a shared secret (symmetric) key without having an existing secure channel. An algorithm that allows this over a public (insecure) channel is called a key-establishment scheme.

Key encapsulation mechanisms (KEMs) are one type of key-establishment scheme, and they're the design of choice for **post-quantum algorithms**. Unlike a [traditional key exchange](../key-exchange.md), the sender's key pair isn't involved, a ciphertext needs to be sent to the recipient, and the shared secret is uniformly random. From the sending side, the algorithm is randomised rather than deterministic.

Here is how it works when doing one trip of communication:

1. **Key generation**: Alice generates a key pair.
2. **Encapsulation**: Bob uses Alice's public key to generate a shared secret and an associated ciphertext. This ciphertext is sent to Alice.
3. **Decapsulation**: Alice uses the ciphertext and her private key to compute the same shared secret.

[X-Wing](https://datatracker.ietf.org/doc/html/draft-connolly-cfrg-xwing-kem) is designed to be the sensible, interoperable **hybrid (post-quantum + traditional security) KEM** for most applications. There are no variants, and it provides a 128-bit security level. The design includes optimisations, combines popular algorithms, and hedges against cryptanalysis advancements.

{% hint style="danger" %}
Private keys **MUST** **NOT** be shared. They **MUST** remain secret.
{% endhint %}

## Usage

### GenerateKeyPair

Fills a span with a randomly generated private key and another span with the associated public key.

```csharp
XWing.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey);
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
XWing.GenerateKeyPair(Span<byte> publicKey, Span<byte> privateKey, ReadOnlySpan<byte> seed);
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
XWing.Encapsulate(Span<byte> sharedSecret, Span<byte> ciphertext, ReadOnlySpan<byte> recipientPublicKey);
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
XWing.Decapsulate(Span<byte> sharedSecret, ReadOnlySpan<byte> ciphertext, ReadOnlySpan<byte> recipientPrivateKey);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`sharedSecret` has a length not equal to `SharedSecretSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`ciphertext` has a length not equal to `CiphertextSize`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`recipientPrivateKey` has a length not equal to `PrivateKeySize`.

[CryptographicException](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicexception)

Invalid ciphertext.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int PublicKeySize = 1216;
public const int PrivateKeySize = 32;
public const int SeedSize = 32;
public const int SharedSecretSize = 32;
public const int CiphertextSize = 1120;
```

## Notes

{% hint style="warning" %}
Not all uses of traditional key exchange can be replaced in a straightforward manner by KEMs. X-Wing doesn't provide non-interactive key exchange (NIKE) or authenticated KEM functionality.
{% endhint %}

{% hint style="warning" %}
Both [X25519](https://elligator.org/) and [ML-KEM](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon) are distinguishable from random (e.g., someone can tell that cryptography is being used from looking at public keys/ciphertexts). Because X-Wing does nothing about this, it's not suitable for scenarios like plausible deniability and censorship-resistance.
{% endhint %}

{% hint style="info" %}
If you [read](https://csrc.nist.gov/pubs/sp/800/227/final) about post-quantum algorithms, you may see the terms 'encapsulation key' and 'decapsulation key'. These mean 'public key' and 'private key' but are specific to KEMs.
{% endhint %}

{% hint style="info" %}
X-Wing uses X25519/ML-KEM-768 for key establishment and SHAKE256/SHA3-256 as KDFs.

The private key is a seed that gets expanded to derive ML-KEM and X25519 key pairs. The public key, ciphertext, and shared secret are the result of concatenating X25519 and ML-KEM-768 values together.
{% endhint %}
