# Padmé

## Purpose

The length of a ciphertext from a [stream cipher](../authenticated-encryption/chacha20-poly1305.md) is equal to the length of the plaintext. Similarly, [block cipher padding](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Padding) only adds a small overhead. This metadata can leak information about what was encrypted and thus is something protocols may want to hide.

[Padmé](https://petsymposium.org/popets/2019/popets-2019-0056.php) is a [popular](https://security.apple.com/blog/imessage-pq3/) deterministic padding scheme with overhead capped to at most +12%, which decreases with input length. It tells you how much padding to add but does not define how that padding should be implemented.

The idea is that multiple input lengths result in the same output length (grouping into buckets), and the number of inputs that map to the same bucket grows. This means the output length hides the exact input length within that range of inputs, preventing padded objects from being uniquely identifiable via their size.

{% hint style="success" %}
Padding should be applied to the plaintext before encryption and removed from the plaintext after decryption. The amount of padding can be stored or reversible padding can be used, like [ISO/IEC 7816-4](iso-iec-7816-4.md).
{% endhint %}

## Usage

### GetPaddedSize

Returns the padded length deterministically calculated from the unpadded length.

```csharp
ulong paddedSize = Padme.GetPaddedSize(ulong unpaddedSize);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`unpaddedSize` is greater than `MaxUnpaddedSize`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

The calculation has resulted in an overflow.

### GetPaddingSize

Returns the difference between the padded length and unpadded length (the amount of padding). This is not for removing padding.

```csharp
ulong paddingSize = Padme.GetPaddingSize(ulong unpaddedSize);
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`unpaddedSize` is greater than `MaxUnpaddedSize`.

[OverflowException](https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception)

The calculation has resulted in an overflow.

## Constants

These are used for validation and/or save you from defining your own constants.

```csharp
public const ulong MaxUnpaddedSize = 18302628885633695744;
```

## Notes

{% hint style="warning" %}
It is very difficult to hide that cryptography is being used. For example, even if padding is done appropriately and there are no readable headers, [X25519](../key-exchange.md) public keys are [distinguishable from random](https://elligator.org/). The same is true for post-quantum KEMs like [ML-KEM](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-kemeleon).

Additionally, metadata around how encrypted data is stored or transmitted can leak information. For instance, file names, directory structures, file timestamps, another protocol using readable headers, etc.
{% endhint %}

{% hint style="warning" %}
Padmé has several limitations to be aware of:

1. There is no padding for very small messages (lengths 0-8).
2. There is minimal padding for small messages. For example, before 33, only two different inputs map to the same output. This gradually improves (4, 8, 16, etc inputs).
3. The deterministic output lengths mean an adversary can potentially fingerprint that Padmé is being used, which helps distinguish a [padded uniform random blob (PURB)](https://en.wikipedia.org/wiki/PURB_\(cryptography\)) from random data. A different bucket also indicates a different message, which can be especially relevant at the boundary between two buckets.
4. If you know the distribution of objects, a dedicated padding function would likely perform better. Padmé was designed for unknown object sizes/as a safe default.

To address these limitations, enforce a minimum length (e.g., 36, 72, or 144) to feed Padmé and consider combining Padmé with randomised padding. However, the latter is easier said than done because a trivial design will mean enough observations can determine the Padmé length (the smallest observed length), which could potentially fingerprint Padmé again.
{% endhint %}

{% hint style="info" %}
The main argument for it being deterministic is that adversaries can cancel out randomised padding with repeated observations to reveal the actual length, which is especially problematic in networking-style scenarios. It also leads to a simple design with predictable overhead.
{% endhint %}

{% hint style="success" %}
Padmé is an improvement on padding to a constant block size because there is no good block size that protects a variety of input lengths, and this often results in significant overhead. Then padding to the next power of two is effective at hiding lengths but can essentially double the original length, which becomes impractical.
{% endhint %}
