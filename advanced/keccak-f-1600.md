# Keccak-f\[1600]

## Purpose

[Keccak-f\[1600\]](https://keccak.team/keccak.html) is the largest variant of the Keccak-f permutation, which is the building block for the [SHA-3](https://csrc.nist.gov/pubs/fips/202/final) [family](https://csrc.nist.gov/pubs/sp/800/185/final) and [KangarooTwelve family](https://datatracker.ietf.org/doc/html/rfc9861) of constructions. It has a 1600-bit state and can be used to construct hash functions, XOFs, MACs, KDFs, stream ciphers, and AEAD schemes.

{% hint style="danger" %}
**This API is more dangerous than the others**. It should only be used by advanced users who want to create [custom constructions](https://eprint.iacr.org/2024/858).

If you don't meet this criteria, please use .NET's [SHA-3](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.sha3_256)/[SHAKE](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.shake128) implementations or wait for [libsodium](https://libsodium.gitbook.io/doc/roadmap) to implement [KangarooTwelve](https://keccak.team/kangarootwelve.html) (KT128/KT256), which should be favoured over all other hash functions/XOFs at the time of writing.
{% endhint %}

## Usage

### IncrementalKeccakf1600

Provides access to the Keccak-f\[1600] permutation. Setting the state to all-zero, absorbing (XORing) data into the state, permuting the state (with 24 or 12 rounds), and squeezing output from the state are supported.

The entire state can be accessed, permuting the state is a separate function, and there's no concept of finalization. This enables flexibility for different custom constructions.

{% hint style="danger" %}
You **MUST** define a secure rate/capacity size. The minimum capacity should be 256 bits for unkeyed hashing/collision resistance scenarios but can be reduced to 192 bits for keyed scenarios (if you don't care about [collision resistance](https://tosc.iacr.org/index.php/ToSC/article/view/11295)). This is because the attacker has less control in keyed modes.

Also, you **MUST** implement proper padding (e.g., [pad10\*1](https://keccak.team/keccak_bits_and_bytes.html) or [pad10\*](https://ascon.isec.tugraz.at/specification.html)) and domain separation (e.g., [XORing a constant](https://competitions.cr.yp.to/round3/norxv30.pdf) after absorbing associated data before absorbing plaintext).

It is possible to do [full-state](https://eprint.iacr.org/2025/2038) [keyed](https://eprint.iacr.org/2023/1520) absorbing/squeezing, but this is not recommended because it's [brittle](https://eprint.iacr.org/2023/1525).
{% endhint %}

```csharp
// Initialize the state to all-zero
using var keccak = new IncrementalKeccakf1600();

// IMPORTANT: Pad/domain separate the message (not shown here)
// Process the message in blocks
foreach (var messageBlock in messageBlocks) {
    // Absorb (XOR) data into the state at offset (default of 0)
    keccak.XorBytes(messageBlock, offset);
    // Permute the state (full or half rounds)
    if (fullRounds) {
        keccak.Permute24(); // Like SHA-3/SHAKE
    }
    else {
        keccak.Permute12(); // Like TurboSHAKE
    }
}

// Squeeze output from the state (once or multiple blocks)
foreach (var outputBlock in output) {
    keccak.ExtractBytes(outputBlock, offset);
    // Permute the state (full or half rounds)
    if (fullRounds) {
        keccak.Permute24(); // Like SHA-3/SHAKE
    }
    else {
        keccak.Permute12(); // Like TurboSHAKE
    }
}

// Reset the state to all-zero
keccak.Reinitialize();
```

#### Exceptions

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`offset` is less than 0 or greater than `StateSize - 1`.

[ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception)

`bytes.Length + offset` is greater than `StateSize`.

[InvalidOperationException](https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception)

Methods cannot be called from multiple threads simultaneously.

[ObjectDisposedException](https://learn.microsoft.com/en-us/dotnet/api/system.objectdisposedexception)

The object has been disposed.

## Constants

These are used for validation and/or save you defining your own constants.

```csharp
public const int StateSize = 200;
```

## Notes

{% hint style="info" %}
Keccak's strengths are explained [here](https://keccak.team/keccak_strengths.html), and a list of third-party cryptanalysis can be found [here](https://keccak.team/third_party.html).
{% endhint %}
