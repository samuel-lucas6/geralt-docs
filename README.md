# Introduction

Geralt is a modern cryptographic library for [.NET 8+](https://dotnet.microsoft.com/en-us/download/dotnet) based on [libsodium](https://doc.libsodium.org/) and inspired by [Monocypher](https://monocypher.org/).

* **Simple**: an easy-to-learn API with descriptive naming. Only one algorithm for each task is provided when possible.
* **Modern**: the latest and greatest cryptographic algorithms, such as AEGIS-128L/AEGIS-256, (X)ChaCha20-Poly1305, BLAKE2b, Argon2id, X25519, and Ed25519.
* **Secure**: libsodium was [audited](https://www.privateinternetaccess.com/blog/libsodium-audit-results/) in 2017 and is the library of choice for [lots](https://doc.libsodium.org/libsodium_users) of projects and [even](https://doc.libsodium.org/libsodium_users#companies-using-libsodium) large companies.
* **Fast**: libsodium is [faster](https://monocypher.org/speed) than many other cryptographic libraries. Furthermore, Geralt uses [Span\<T>](https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/connect/csharp-all-about-span-exploring-a-new-net-mainstay) buffers to avoid memory allocations.

## Installation

Geralt is available as a [NuGet](https://www.nuget.org/packages/Geralt) package. It's supported on the following [platforms](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog):

| Windows     | Linux            | macOS       | Other         |
| ----------- | ---------------- | ----------- | ------------- |
| `win-x64`   | `linux-x64`      | `osx-x64`   | `ios`         |
| `win-x86`   | `linux-musl-x64` | `osx-arm64` | `tvos`        |
| `win-arm64` | `linux-arm64`    |             | `maccatalyst` |
|             | `linux-arm`      |             | `android`     |

[Stable releases](https://doc.libsodium.org/doc/quickstart#what-is-the-difference-between-point-releases-and-stable-releases) of the [libsodium NuGet package](https://www.nuget.org/packages/libsodium/) are supported without Geralt being updated, whereas point releases require Geralt to be updated.

{% hint style="info" %}
Note that libsodium requires the [Visual C++ Redistributable for Visual Studio 2015-2022](https://learn.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170) on <mark style="color:yellow;">**Windows**</mark>. Instructions on how to deal with this can be found [here](getting-libsodium-to-work-on-windows.md).

To get Geralt working on <mark style="color:yellow;">**Android**</mark>, you need to [build the libsodium binaries yourself](https://doc.libsodium.org/installation#cross-compiling-to-android) because they aren't included in the libsodium NuGet package. You then need to [modify](https://github.com/ektrah/nsec/issues/81) the project slightly to target that platform.
{% endhint %}

## Source code

You can find the source code on [GitHub](https://github.com/samuel-lucas6/Geralt).

As of v3.3.0, this documentation is also [mirrored](https://github.com/samuel-lucas6/geralt-docs) to GitHub.

## License

Geralt is licensed under the [MIT](https://github.com/samuel-lucas6/Geralt/blob/main/LICENSE) license.

## Contact

To report a bug, provide feedback, or ask for a new feature, please raise a [GitHub issue](https://github.com/samuel-lucas6/Geralt/issues/new).

For questions and technical support, please create a [GitHub discussion](https://github.com/samuel-lucas6/Geralt/discussions/new/choose).

Finally, please see the [SECURITY.md](https://github.com/samuel-lucas6/Geralt/blob/main/SECURITY.md) file on GitHub for vulnerability reporting.

## Goals

* [Span\<T>](https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/connect/csharp-all-about-span-exploring-a-new-net-mainstay) all the things: enables the secure erasure of bytes/chars and boosts performance.
* Descriptive naming: BLAKE2b, not GenericHash.
* Same vocabulary for everything: key, nonce, salt, input keying material, output keying material, etc.
* Minimal parameters: no key parameter for an unkeyed hash.
* Consistent parameter ordering: buffers come first.
* Public constants: easy to create buffers.
* One algorithm for each task: (X)ChaCha20-Poly1305, BLAKE2b, Argon2id, X25519, and Ed25519.
* Some low-level functions: useful for [custom](https://github.com/jedisct1/libsodium-xchacha20-siv) [constructions](https://github.com/jedisct1/spake2-ee).
* Internal secure zeroing: avoids leaving sensitive data in temporary buffers.

## Out of scope

* Full misuse resistance (e.g. no nonces or optional nonces). This can limit the user, doesn't work well with spans, and overcomplicates code.
* Solving the key reuse problem (e.g. a [mandatory context](https://youtu.be/1NVgRPb0tHU) for everything or [wrappers](https://nsec.rocks/docs/api/nsec.cryptography.key) instead of raw bytes). I'm [not](https://github.com/ektrah/nsec/issues/31) convinced either tactic works, and it again adds complexity.
* Old [NaCl](https://nacl.cr.yp.to/) APIs, such as `crypto_box`. These [shouldn't](https://github.com/jedisct1/libsodium/issues/586) be used.
* Other primitives unless they solve a problem. AES-GCM causes problems (e.g. it [requires](https://doc.libsodium.org/secret-key_cryptography/aead/aes-256-gcm#limitations) hardware support). [AEGIS](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-aegis-aead) solves problems (e.g. it's [key committing](https://eprint.iacr.org/2022/268) and supports random nonces whilst being [faster](https://eprint.iacr.org/2023/523)/[stronger](https://competitions.cr.yp.to/round3/aegisv11.pdf) than AES-GCM).
* Experimental ideas/custom constructions (e.g. anything without an RFC or Internet-Draft), which can go in a [separate project](https://github.com/samuel-lucas6/Daence.NET).
* Duplicate methods that return byte arrays.
* Unnecessary 'convenience' functions, like `GenerateKey()` in almost every class.
* Internal [guarded heap allocations](https://doc.libsodium.org/memory_management#guarded-heap-allocations), which [reduce](https://github.com/ektrah/nsec/issues/52) performance and are unnecessary for very short-lived secrets.
* Support for old/no longer supported versions of [.NET](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) and [.NET MAUI](https://dotnet.microsoft.com/en-us/platform/support/policy/maui).

## Acknowledgements

I'd like to say a big thanks to:

* [GitBook](https://www.gitbook.com/) for their free open source plan.
* [Tuta](https://tuta.com/) for donating their private email service.
* [Frank Denis](https://github.com/jedisct1) for writing the [libsodium](https://doc.libsodium.org/) library.
* [Loup Vaillant](https://github.com/LoupVaillant) for writing the [Monocypher](https://github.com/LoupVaillant/Monocypher) library.
* [Klaus Hartke](https://github.com/ektrah) for creating [NSec](https://nsec.rocks/) and doing .NET [PRs](https://github.com/jedisct1/libsodium/commits?author=ektrah) for libsodium.
* [Trond Arne Bråthen](https://github.com/tabrath) for creating the [libsodium-core](https://github.com/ektrah/libsodium-core) library.​
* [Adam Caudill](https://github.com/adamcaudill) and everyone who contributed to the [libsodium-net](https://web.archive.org/web/20221205225204/https://github.com/adamcaudill/libsodium-net) library.​
* Everyone who has [contributed](https://github.com/samuel-lucas6/Geralt/graphs/contributors) to, used, or provided feedback about Geralt.
