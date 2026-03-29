---
hidden: true
---

# Getting libsodium to work on Windows

libsodium requires the [latest Microsoft Visual C++ Redistributable](https://learn.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170) on Windows. This dependency is included in the .NET SDK. When publishing software, there are three ways to deal with this:

1. Install this as part of your application setup/as a [package manager dependency](https://docs.chocolatey.org/en-us/create/package-dependencies/).
2. Ask the user to manually install this.
3. Bundle the `vcruntime140.dll` file with your executable.

If you want your program to be portable (e.g., [self-contained](https://learn.microsoft.com/en-us/dotnet/core/deploying/?pivots=visualstudio#self-contained-deployment)), you have to take the third approach. This can be done using the following steps:

1. Download the `VisualCppRedist_AIO_x86_x64.exe` file from the [latest release](https://github.com/abbodi1406/vcredist/releases) of [this](https://github.com/abbodi1406/vcredist) GitHub repo.
2. Use [7-Zip](https://www.7-zip.org/) to extract the downloaded executable (e.g., right click, hover over 7-Zip, and click Extract to "VisualCppRedist\_AIO\_x86\_x64\\").
3. In the extracted folder, navigate to `2022`, `x86` and/or `x64` (depending on which platform your application targets), `System` or `System64`, and copy the `vcruntime140.dll` file(s) to your project folder.
4. Add the `vcruntime140.dll` file(s) as [embedded resources](https://stackoverflow.com/questions/4111160/resources-where-to-put-them-and-how-to-reference-them-in-c-sharp) in your project.
5. Write some code that extracts the relevant (x86 or x64) file to the [location of your executable](https://github.com/samuel-lucas6/Kryptor/blob/v4.1.0/src/Kryptor/Program.cs#L148) or to the [directory where the `libsodium.dll` file is located](https://github.com/samuel-lucas6/Kryptor/commit/57e72364b865ffcdf22d9050eb711b6b873b3277) when your application starts.
6. Test that libsodium in your application works/doesn't throw a `PlatformNotSupportedException` on a Windows machine that doesn't have the Visual C++ Redistributable installed (e.g., in [Windows Sandbox](https://learn.microsoft.com/en-us/windows/security/application-security/application-isolation/windows-sandbox/windows-sandbox-overview) or a [virtual machine](https://www.virtualbox.org/)).
