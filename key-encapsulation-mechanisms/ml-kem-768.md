# ML-KEM-768

## Purpose

When communicating with another party, you often need a way to establish a shared secret (symmetric) key without having an existing secure channel. An algorithm that allows this over a public (insecure) channel is called a key-establishment scheme.

Key encapsulation mechanisms (KEMs) are one type of key-establishment scheme, and they're the interface of choice for **post-quantum algorithms**, which protect against cryptographically relevant quantum computers (CRQCs). Even without a CRQC today, an adversary can capture encrypted data that relies on [traditional key-establishment algorithms](../key-exchange.md) ready for decryption when such a CRQC becomes available (a [store now, decrypt later attack](https://en.wikipedia.org/wiki/Harvest_now%2C_decrypt_later)).

Unlike a traditional key exchange, the sender's key pair isn't involved, a ciphertext needs to be sent to the recipient, and the shared secret is uniformly random. From the sending side, the algorithm is randomised rather than deterministic (the sender doesn't choose the shared secret).

Here is how it works when doing one trip of communication (real protocols often do multiple trips):

1. **Key generation**: the recipient generates a key pair and shares the public key with the sender.
2. **Encapsulation**: the sender uses the recipient's public key to generate a shared secret and an associated ciphertext. This ciphertext is sent to the recipient.
3. **Decapsulation**: the recipient uses the ciphertext and their private key to compute the same shared secret.

[ML-KEM-768](https://csrc.nist.gov/pubs/fips/203/final) is the middle (192-bit) security strength variant of ML-KEM, which is one of the algorithms standardised by [NIST](https://csrc.nist.gov/Projects/post-quantum-cryptography/post-quantum-cryptography-standardization/selected-algorithms) as part of the Post-Quantum Cryptography (PQC) Standardization (competition) process. This variant provides protection against cryptanalysis advancements compared to ML-KEM-512 whilst being [lighter](https://pqshield.github.io/nist-sigs-zoo/kems/?s=ML-KEM) (smaller parameters/marginally faster) than ML-KEM-1024. All variants are [faster](https://pqshield.github.io/nist-sigs-zoo/kems/?s=ML-KEM%2CECDH) than traditional key exchange algorithms (e.g., X25519) but have larger parameters.

{% hint style="danger" %}
Private keys **MUST** **NOT** be shared. They **MUST** remain secret and be protected from modification.
{% endhint %}

{% hint style="success" %}
Consider using [X-Wing](x-wing.md) instead, which uses X25519 + ML-KEM-768 to provide [classical security still](https://soatok.blog/2022/01/27/the-controversy-surrounding-hybrid-cryptography/) if ML-KEM-768 is unexpectedly broken. Whilst this is [arguably unnecessary](https://soatok.blog/2026/04/13/hybrid-constructions-the-post-quantum-safety-blanket/) and has [some drawbacks](https://www.ncsc.gov.uk/paper/next-steps-in-preparing-for-post-quantum-cryptography#section_5), it is a [popular/recommended approach](https://crypto.stackexchange.com/q/119797).

However, X-Wing isn't suitable in all cases, like when doing protocols requiring plausible deniability or censorship-resistance. In such cases, one may need to create their own hybrid KEM.
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

{% hint style="danger" %}
A KEM does **NOT** provide authentication of either party. It's important to verify that any public keys and ciphertexts came from who you expect, which is typically done via [digital signatures](../digital-signatures.md).
{% endhint %}

{% hint style="danger" %}
One would expect that different ciphertexts/public keys produce different shared secrets. However, an adversary that can control a **private key** can cause the [same shared secret](https://eprint.iacr.org/2024/523) to be generated.

Technically, only protocols where the private key cannot be fully trusted (e.g., it's received from a third party or sometimes revealed) are at risk. **However, it's best practice to hash ciphertexts/public keys alongside shared secrets during key derivation**, which mitigates this.

Alternatively, one can store a random 256-bit seed as the private key and use a KDF to expand it to 512 bits for ML-KEM seeded key generation to derive the large private key, as done in [X-Wing](x-wing.md). This prevents a malformed private key because the attacker can't control the key derivation.
{% endhint %}

{% hint style="warning" %}
Not all uses of traditional key exchange can be replaced in a straightforward manner by KEMs. For example, KEMs don't provide [non-interactive key exchange (NIKE)](https://eprint.iacr.org/2023/271) functionality (where both users can compute the shared secret without interaction if they know each other's public key).

KEMs are inherently synchronous and interactive because of the ciphertext. This [prevents](https://eprint.iacr.org/2022/539) non-interactive authentication via static public keys, which avoids digital signatures. In an offline context, the best you can do is encapsulate to the recipient's static public key (no sender keys can be involved because that would require an interactive protocol).
{% endhint %}

{% hint style="warning" %}
ML-KEM public keys and ciphertexts are [distinguishable from random](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon) (e.g., someone can tell that cryptography is being used). Therefore, it's not suitable for protocols requiring plausible deniability or censorship-resistance without using a scheme such as [Kemeleon](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon), which is not implemented in libsodium.
{% endhint %}

{% hint style="success" %}
Static (long-term) as well as ephemeral public keys can be used safely. Ephemeral public keys help provide [forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy), which protects prior communications in the event of a key compromise.
{% endhint %}

{% hint style="success" %}
ML-KEM has a [tiny](https://csrc.nist.gov/pubs/fips/203/final) probability of decapsulation failure. Even if everything is done honestly/correctly, both parties may not derive the same shared secret.

However, practically speaking, this will [never happen](https://datatracker.ietf.org/doc/html/draft-sfluhrer-cfrg-ml-kem-security-considerations). In other words, this isn't something to worry about.
{% endhint %}

{% hint style="success" %}
If an attacker substitutes a public key or modifies/replaces a ciphertext, the derived shared secret will be different between the two parties, which will cause an error in any properly designed protocol.
{% endhint %}

{% hint style="info" %}
Encapsulation can error due to a check that the integers encoded in the public key are in the valid range. This sort of check is not performed for decapsulation.
{% endhint %}

{% hint style="info" %}
Whilst the shared secret is 256 bits long, its security strength is actually 192 bits due to the security level of ML-KEM-768.
{% endhint %}

{% hint style="info" %}
If you [read](https://csrc.nist.gov/pubs/sp/800/227/final) about post-quantum algorithms, you may see the terms 'encapsulation key' and 'decapsulation key'. These mean 'public key' and 'private key' but are specific to KEMs.
{% endhint %}
