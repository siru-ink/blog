+++
title = "On Code Signing for MacOS and Windows"
date = "2025-10-31"
+++
## Intro to Code Signing

Most modern operating systems (apart from Linux of course), have added in
programs that prevent applications from unknown sources to be run directly. For
Apple this is called the Gatekeeper[^1] program, whereas on Windows it is known
as SmartScreen[^2]. While these efforts in theory could help prevent the
proliferation of viruses, in my humble opinion they have mainly been used to
gatekeep platforms and force developers to pay for official signing
certificates, while actual malicious programs can still get around these
protections fairly easily.

## For MacOS

The correct way to prevent an app from being quarantined by the Gatekeeper
program is of course to pay for an Apple developer ID[^3], then create a
developer signing certificate, and then include this certificate in the build
process for the application bundle. For example if using
[PyInstaller](https://pyinstaller.org/en/stable/), this can be done using the
`--codesign-identity` [CLI](https://en.wikipedia.org/wiki/Command-line_interface)
option.

### Manually Dequarantine an App

However, there is also the less UX friendly approach of simply overriding
Apple's gatekeeper on the target device. This can only be done after the
applications has already been installed on the target device, and an
unsuccessful launch of the app was attempted. When Gatekeeper prevents the
running of an application, it sets an extra quarantine attribute on the
application bundle. This attribute can be seen via the command:

```bash
xattr -l /path/to/application.app
```

Using the same CLI utility, this quarantine flag can then also be removed using
the following command:

```bash
xattr -dr com.apple.quarantine /path/to/application.app
```

which recursively deletes the extra attribute from all the files in the
application bundle. This allows the app to be run normally on future calls.

> [!Note]
> It should however be noted, that this also means if the app contained actual
> harmful code, this would allow the harmful code to run on the computer in the
> future.

## For Windows

The process is largely similar to how MacOS functions. The code still needs to
be officially signed by a certificate. However, the certificate can be purchased
from other sources than just Microsoft itself. For example, (at least at time of
writing) [Digicert](https://www.digicert.com/) is a trusted root
[certificate authority](https://en.wikipedia.org/wiki/Certificate_authority)[^4]
and they sell code signing certificates, such as
[here](https://www.digicert.com/signing/code-signing-certificates).

This certificate can then be used to sign the code using Microsoft's `signtool`,
which is part of the Windows SDK, using the following command (assuming the
certificate was already installed on the machine being used for signing):[^5]

```powershell
signtool sign /a /fd SHA256 /tr http://timestamp.digicert.com `
  /td SHA256 MyFile.exe
```

> [!Note]
> It is apparently also possible to install just the `signtool` without
> installing the entire Windows SDK, but this is not the standard procedure.
> For more info see [this](https://stackoverflow.com/a/52963704/31726834)
> StackOverflow post.

To my knowledge, there is no way to manually remove the SmartScreen flag on the
file, however (at least as of right now) it is still possible to bypass the
dialog and continue to run the app even when it is flagged on Windows.

## What About Linux?

The answer is simple. Linux will let you run anything without complaint.

[^1]: See more: [Sign Mac Software for Gatekeeper](https://developer.apple.com/developer-id/)
[^2]: See more: [SmartScreen by Microsoft](https://learn.microsoft.com/en-us/windows/security/operating-system-security/virus-and-threat-protection/microsoft-defender-smartscreen/)
[^3]: For reference: [Developer Account](https://developer.apple.com/programs/whats-included/) At time of writing this is a $99/year charge.
[^4]: As far as I know this list rarely changes, but more information can be found [here](https://learn.microsoft.com/en-us/security/trusted-root/participants-list).
[^5]: See more: [StackOverflow](https://stackoverflow.com/questions/252226/)
