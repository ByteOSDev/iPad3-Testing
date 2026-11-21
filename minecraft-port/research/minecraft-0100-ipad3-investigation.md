# Minecraft PE 0.10.0 on iPad 3 — Research Log

**Project:** iPad 3 Compatibility Research  
**Repository:** `ByteOSDev/iPad3-Testing`  
**Target device:** iPad 3,1 (Wi-Fi)  
**Target application:** Minecraft: Pocket Edition 0.10.0  
**Current installed version:** Minecraft PE 0.9.5  
**Research status:** In progress — IPA installation failure under investigation  
**Last updated:** November 21, 2026

---

## 1. Research objective

The goal is to determine whether a newer genuine version of Minecraft: Pocket Edition can run on the iPad 3,1.

The target is the original Minecraft application, not a recreation, clone, or replacement.

The iPad 3 currently runs Minecraft PE 0.9.5. The investigation began after finding historical information suggesting that Minecraft PE 0.10 introduced 64-bit support.

An important question was whether 0.10.0 dropped 32-bit ARM support or added ARM64 while retaining ARMv7 compatibility.

The specific 0.10.0 IPA being investigated has been confirmed to contain both ARMv7 and ARM64 architectures.

---

## 2. Target device specifications

Confirmed device information:

| Property | Value |
|---|---|
| Device | iPad 3,1 |
| Model | Wi-Fi iPad 3 |
| Board identifier | J1AP |
| Processor | Apple A5X |
| SoC identifier | S5L8945X |
| CPU architecture | ARMv7, 32-bit |
| RAM | 1,034,944,512 bytes (~1 GB) |
| Current iOS | iOS 9.3.5 |
| Build | 13G36 |
| Darwin | 15.6.0 |
| XNU | 3248.61.1 |

The device has a 32-bit ARM processor. It cannot execute an ARM64-only application.

The presence of an ARMv7 executable slice is therefore essential.

### Jailbreak and access

- The iPad has been prepared using a checkm8-a5-based jailbreak setup.
- The device boots normally.
- SSH access as `root` works.
- The user has intentionally avoided using ramdisk SSH for this investigation.
- Filza is available for on-device file management and installation attempts.

Previously used device IP: `10.0.0.142`  
This address may change.

Successful SSH command:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa root@10.0.0.142
```

---

## 3. Mac research environment

The research is being conducted on an M1 MacBook Air.

Tools used include:

- Terminal
- `unzip`
- `plutil`
- `otool`
- `codesign`
- Filza on the iPad
- AppInst on the iPad
- GitHub repository `ByteOSDev/iPad3-Testing`

The current Mac working directory has been:

```text
~/Desktop/MC170-Inspection
```

The 0.10.0 IPA is located at:

```text
~/Downloads/minecraft 0.10.0.ipa
```

Extracted inspection directory:

```text
/tmp/mc010-check/
```

---

## 4. Historical architecture investigation

The initial question was whether Minecraft PE 0.10.0 introduced ARM64 support at the expense of ARMv7.

Historical version records and development information were investigated.

The research indicates that Minecraft PE 0.10.0 added a 64-bit iOS build while ARMv7 builds continued to exist.

This distinction matters because the iPad 3 cannot run ARM64-only applications.

Historical research references from earlier investigation:

- `turn927062view1` — iOS Obscura development/build record documenting the addition of a 64-bit iOS build.
- `turn927062view0` — iOS Obscura version record listing Minecraft PE 0.10.0 variants, including ARMv7 + ARM64 and ARMv7-only copies.
- `turn464600search10`
- `turn589679search0`
- `turn589679search2`
- `turn589679search1`
- `turn685892view0`

**Conclusion:** The version number alone does not establish compatibility. The actual executable inside the IPA must be inspected.

---

## 5. Separate Minecraft 1.7.0 investigation

A different IPA, named `minecraft 1.7.0.ipa`, was investigated earlier.

It must not be confused with the current 0.10.0 IPA.

### 1.7.0 metadata

| Property | Value |
|---|---|
| Bundle identifier | `com.mojang.minecraftpe` |
| Version | `1.7.0` |
| Build | `1.7.0.13` |
| Minimum OS | 9.0 |
| SDK | iPhoneOS 11.3 |
| Xcode | 9.3 |
| Executable | `minecraftpe` |
| Architecture | ARM64 only |

The executable was identified as a 64-bit ARM Mach-O binary.

It did not contain an ARMv7 slice.

The recorded device support information also did not include the iPad 3.

**Conclusion:** The investigated 1.7.0 IPA is not compatible with the iPad 3's ARMv7 processor.

Earlier research log:

```text
minecraft-port/logs/ipa-inspection-1.txt
```

Recorded GitHub file SHA:

```text
0b5827226bdb6f6bee1963941c65c36b793ea57d
```

The Dropbox source previously investigated was named `minecraft 1.7.0.ipa`.

---

## 6. Current Minecraft 0.10.0 IPA

The current investigation concerns:

```text
~/Downloads/minecraft 0.10.0.ipa
```

This is a separate IPA from the 1.7.0 file.

### Confirmed application metadata

| Property | Value |
|---|---|
| Version | 0.10.0 |
| Bundle identifier | `com.mojang.minecraftpe` |
| Minimum iOS | 5.1.1 |
| Architectures | ARMv7 + ARM64 |
| Device family | Includes iPad |
| Main executable | `minecraftpe` |
| Executable size | 8,825,008 bytes |

The IPA contains the expected application bundle:

```text
Payload/minecraftpe.app/
```

Important files found:

```text
Payload/minecraftpe.app/Info.plist
Payload/minecraftpe.app/minecraftpe
Payload/minecraftpe.app/SC_Info/minecraftpe.supf
Payload/minecraftpe.app/SC_Info/minecraftpe.sinf
Payload/minecraftpe.app/SC_Info/minecraftpe.supp
Payload/minecraftpe.app/SC_Info/Manifest.plist
Payload/minecraftpe.app/_CodeSignature/CodeResources
```

The package includes the expected application resources and metadata.

### 6.1 ZIP archive integrity

Command:

```bash
unzip -t ~/Downloads/"minecraft 0.10.0.ipa" | tail -5
```

Output:

```text
testing: Payload/minecraftpe.app/silverfish_hit2.m4a OK
testing: Payload/minecraftpe.app/grass4.pcm OK
testing: Payload/minecraftpe.app/glyph_AD.png OK
testing: Payload/minecraftpe.app/sheep_4.png OK
No errors detected in compressed data of /Users/alisonrinder/Downloads/minecraft 0.10.0.ipa.
```

**Result:** The ZIP archive passed its integrity test.

### 6.2 Info.plist inspection

Command:

```bash
unzip -p ~/Downloads/"minecraft 0.10.0.ipa" Payload/minecraftpe.app/Info.plist | head -c 100
```

The output began with:

```text
bplist00
```

This is the normal header for a binary Apple property list.

The plist was extracted separately:

```bash
unzip -p ~/Downloads/"minecraft 0.10.0.ipa" Payload/minecraftpe.app/Info.plist > /tmp/mc010-Info.plist
plutil -lint /tmp/mc010-Info.plist
```

Output:

```text
/tmp/mc010-Info.plist: OK
```

The entire IPA was then extracted:

```bash
rm -rf /tmp/mc010-check
mkdir /tmp/mc010-check
unzip -q ~/Downloads/"minecraft 0.10.0.ipa" -d /tmp/mc010-check
plutil -lint /tmp/mc010-check/Payload/minecraftpe.app/Info.plist
```

Output:

```text
/tmp/mc010-check/Payload/minecraftpe.app/Info.plist: OK
```

**Result:** The Info.plist is valid both when read directly from the IPA and after extraction.

---

## 7. ARMv7 executable inspection

The executable was inspected using:

```bash
otool -l -arch armv7 /tmp/mc010-check/Payload/minecraftpe.app/minecraftpe | grep -A5 LC_ENCRYPTION_INFO
```

Output:

```text
cmd LC_ENCRYPTION_INFO
cmdsize 20
cryptoff 16384
cryptsize 2392064
cryptid 1
```

### Interpretation

The ARMv7 executable contains an `LC_ENCRYPTION_INFO` load command.

The reported `cryptid` is `1`, indicating that the executable's relevant segment is marked as encrypted.

This is an important finding for understanding the executable's state.

However, this result does **not** independently establish why the iPad's installer reports an Info.plist loading failure.

The encryption result and the installation error must be investigated as separate issues unless further evidence connects them.

No DRM decryption or circumvention has been attempted.

---

## 8. Installation attempt on the iPad

The 0.10.0 IPA was sent to the iPad and an installation was attempted using AppInst through Filza.

The installation failed.

### Exact AppInst output

```text
appinst (App Installer)
Copyright (C) 2014-2024 Karen/あけみ

Initialising appinst installation session ID emROhtqFD7TBF8RqAQ3pWPoGRRYJbdXC (PID 8816)…

Installing "com.mojang.minecraftpe"…

An error occurred while attempting to install the app!

NSError info: Error Domain=LaunchServicesError Code=0 "(null)" UserInfo={Error=PackageInspectionFailed, ErrorDescription=Failed to load Info.plist from bundle at path /private/var/installd/Library/Caches/com.apple.mobile.installd.staging/temp.iImUWe/extracted/Payload/minecraftpe.app}

Cleaning up appinst session ID emROhtqFD7TBF8RqAQ3pWPoGRRYJbdXC (PID 8816)…

Deleting temporary directory…

Failed to install "com.mojang.minecraftpe".
```

### Error interpretation

The reported error is:

```text
PackageInspectionFailed
```

The description says:

```text
Failed to load Info.plist from bundle
```

This is notable because the same IPA's Info.plist has already passed `plutil -lint` both before and after extraction on the Mac.

Therefore, the plist's basic binary format and syntax are not currently the leading explanation.

Possible areas to investigate include:

- How the installer extracts and stages the IPA.
- Whether the staged bundle has the expected files and directory structure.
- Whether file attributes or permissions are preserved.
- Whether the installer rejects some package metadata.
- Whether signature or package-validation checks are failing.
- Whether the device's `installd` logs contain a more specific underlying error.

These are hypotheses, not confirmed causes.

**The app has not successfully installed.**

---

## 9. Code-signature inspection

The extracted app bundle was inspected with:

```bash
codesign -dv --verbose=4 /tmp/mc010-check/Payload/minecraftpe.app 2>&1
```

Exact output:

```text
Executable=/private/tmp/mc010-check/Payload/minecraftpe.app/minecraftpe

Identifier=com.mojang.minecraftpe

Format=app bundle with Mach-O universal (armv7 arm64)

CodeDirectory v=20200 size=23446 flags=0x0(none) hashes=1163+5 location=embedded

VersionPlatform=2

VersionMin=458752

VersionSDK=524544

Hash type=sha1 size=20

CandidateCDHash sha1=cf50cb522d4c8fde56aea22404fd8c207acc994d

CandidateCDHashFull sha1=cf50cb522d4c8fde56aea22404fd8c207acc994d

Hash choices=sha1

CMSDigest=a5913da2e09ab897a1f59d19727cf95308aa6fdba3a9676ad253eb09d49b10fd

CMSDigestType=2

Page size=4096

CDHash=cf50cb522d4c8fde56aea22404fd8c207acc994d

Signature size=3487

Authority=Apple iPhone OS Application Signing

Authority=Apple iPhone Certification Authority

Authority=Apple Root CA

Info.plist entries=35

TeamIdentifier=HR992ZEAE6

Sealed Resources version=2 rules=12 files=737

Internal requirements count=1 size=104

Total signatures=1

Chosen signature=1
```

### Interpretation

The output establishes that:

- The app bundle is recognized by macOS as a signed application bundle.
- The executable is recognized as a universal Mach-O containing ARMv7 and ARM64.
- The bundle identifier is `com.mojang.minecraftpe`.
- A code signature is present.
- The signature metadata identifies an Apple iPhone OS Application Signing authority chain.
- The signature reports 737 sealed resource files.
- The bundle's signature metadata includes 35 Info.plist entries.

This output is from `codesign -dv`, which displays signature information. It is not, by itself, a complete signature verification result on the target iPad.

### 9.1 _CodeSignature directory

Command:

```bash
ls -la /tmp/mc010-check/Payload/minecraftpe.app/_CodeSignature
```

Output:

```text
total 232

drwxr-xr-x@   3 alisonrinder  wheel      96 Nov 21 14:39 .
drwxr-xr-x@ 730 alisonrinder  wheel   23360 Nov 21 14:39 ..

-rw-r--r--@   1 alisonrinder  wheel  116519 Nov 15  2014 CodeResources
```

**Result:** The `_CodeSignature` directory exists and contains `CodeResources`.

The file is dated November 15, 2014, consistent with an older application package.

---

## 10. Minecraft world backup

Before attempting to install a newer version, the existing Minecraft worlds were backed up.

The original application data location was identified as:

```text
Documents/games/com.mojang/minecraftWorlds
```

The user located the `games` and `com.mojang` folders in Filza.

The entire `com.mojang` folder was copied somewhere outside Minecraft's application data container.

The user confirmed completion.

**Backup status:** User reports that the backup is complete.

The backup should remain untouched during testing.

Installing an update with the same bundle identifier may preserve application data, but this is not guaranteed in every installation scenario. A newer Minecraft version may also modify world data when opening an existing world.

Do not uninstall Minecraft 0.9.5 as part of the current investigation.

Do not use the only backup copy as the test copy.

---

## 11. Confirmed findings versus unresolved questions

### Confirmed

- The target iPad 3,1 uses an ARMv7 32-bit A5X processor.
- The current installed Minecraft version is 0.9.5.
- The investigated 1.7.0 IPA is ARM64-only and is incompatible with the target processor.
- The investigated 0.10.0 IPA contains ARMv7 and ARM64 slices.
- The 0.10.0 IPA's ZIP archive passes its integrity test.
- The 0.10.0 Info.plist passes `plutil -lint`.
- The extracted Info.plist also passes `plutil -lint`.
- The ARMv7 executable reports `cryptid 1`.
- The extracted app bundle has recognizable code-signature metadata.
- The `_CodeSignature/CodeResources` file exists.
- AppInst installation fails with `PackageInspectionFailed`.
- The user has backed up the existing `com.mojang` data folder.

### Not yet confirmed

- Why LaunchServices reports that it cannot load Info.plist when the Mac can parse it successfully.
- Whether the app's signature passes a full verification check.
- Whether the iPad installer is rejecting the signature, package structure, or another property.
- Whether the staged bundle on the iPad contains the same files as the original IPA.
- Whether the application can launch on the iPad after a legitimate successful installation.
- Whether all of Minecraft 0.10.0's runtime dependencies are compatible with iOS 9.3.5 on the A5X.
- Whether Minecraft 0.10.4 or another later 0.10.x release would be a better candidate.

---

## 12. Next diagnostic steps

The next steps should focus on identifying the installation failure without modifying the original IPA or bypassing DRM.

### Step 1 — Full signature verification on the Mac

Run:

```bash
codesign --verify --deep --strict --verbose=4 /tmp/mc010-check/Payload/minecraftpe.app
```

Record the complete output.

A failure should be documented rather than immediately treated as proof that the IPA is unusable. Older App Store signatures and modern macOS verification behavior may differ.

### Step 2 — Check executable file permissions

Run:

```bash
ls -l /tmp/mc010-check/Payload/minecraftpe.app/minecraftpe
```

Record the file's permissions, ownership, and size.

### Step 3 — Inspect the full Info.plist

Run:

```bash
plutil -p /tmp/mc010-check/Payload/minecraftpe.app/Info.plist
```

Record the full output, especially:

- `CFBundleIdentifier`
- `CFBundleExecutable`
- `CFBundleVersion`
- `CFBundleShortVersionString`
- `MinimumOSVersion`
- `UIDeviceFamily`
- `LSRequiresIPhoneOS`

### Step 4 — Inspect iPad installation logs

If possible, capture the device's `installd` and LaunchServices logs around another controlled installation attempt.

The goal is to find a more specific reason for `PackageInspectionFailed`.

Do not repeatedly reinstall or alter the current Minecraft installation without a reason.

### Step 5 — Keep the original package unchanged

Preserve:

```text
~/Downloads/minecraft 0.10.0.ipa
```

Use separate copies for any non-destructive packaging experiments.

Do not decrypt, patch, or otherwise circumvent FairPlay DRM.

If the package cannot be installed through a legitimate supported method, investigate an appropriately licensed and signed build instead.

---

## 13. Important distinction: architecture compatibility versus installation compatibility

The current research has established that the 0.10.0 IPA contains an ARMv7 executable slice.

That is a positive architecture-level finding.

It does not establish that the app can be installed or run successfully.

There are three separate questions:

1. **Architecture:** Does the executable contain ARMv7 code?  
   Current answer: Yes.

2. **Installation:** Does iOS accept the IPA and install the application bundle?  
   Current answer: No. AppInst reports `PackageInspectionFailed`.

3. **Runtime:** If installation succeeds, can the application actually launch and operate on the iPad 3?  
   Current answer: Not tested.

These questions must remain separate in future research notes.

---

## 14. Current overall status

**Minecraft PE 0.10.0 is a promising compatibility candidate at the architecture level, but installation is currently blocked.**

The IPA is structurally readable on macOS, its Info.plist parses successfully, and its executable is recognized as universal ARMv7/ARM64.

The current blocker is the iPad's installation failure:

```text
PackageInspectionFailed
Failed to load Info.plist from bundle
```

The latest evidence shows that the extracted application bundle has code-signature metadata and a `_CodeSignature/CodeResources` file.

The next useful investigation is full signature verification, executable permissions, complete Info.plist inspection, and the iPad's installation logs.

**No successful installation or launch of Minecraft 0.10.0 has been recorded yet.**
