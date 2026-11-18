# Public iPad 3 iOS 10 Port Research Addendum

**Date:** 2026-11-17  
**Scope:** Public, non-device-sensitive research only. No firmware was downloaded or installed during this review.

## Proposed donor IPSW

The supplied URL identifies **iPad3,4**, the Wi-Fi iPad (4th generation), running **iOS 10.3.3 build 14G60**:

- Filename: `iPad_32bit_10.3.3_14G60_Restore.ipsw`
- Reported size: 1.8 GiB
- Reported SHA-256: `3e2bd7d18823d93736a68d85961ff7ece5dd932a19c58d52e43282a6ea6ac312`
- Release date: 2017-07-19
- Source: [IPSW Downloads entry](https://ipsw.me/download/iPad3,4/14G60/)

The page reports the firmware as signed, but signing status is time-sensitive and must be rechecked immediately before any restore operation. This project does not authorize a restore.

## Hardware and identifier boundary

The donor and target are different products:

| Role | Product | Board | Application processor |
|---|---|---|---|
| Target | iPad3,1 | J1AP | Apple A5X |
| Donor | iPad3,4 | P101AP | Apple A6X |

Sources: [iPad3,1 reference](https://theapplewiki.com/wiki/IPad3%2C1), [iPad3,4 reference](https://theapplewiki.com/wiki/P101AP).

The public evidence therefore supports using the iPad3,4 IPSW as a **research donor**, not as a directly installable iPad3,1 firmware. Its kernelcache, device tree, boot components, board configuration, and hardware-specific firmware are expected to be identified for P101AP/S5L8955X rather than J1AP/S5L8945X. Replacing those components blindly would not constitute a validated port and could make the target unbootable.

## Safe use of the IPSW as a research base

If the IPSW is obtained independently, the safe analysis sequence is:

1. Verify the downloaded file against the published SHA-256.
2. Treat the IPSW as an archive and inventory its manifest and component names.
3. Extract files only into a new analysis directory; preserve the original archive.
4. Compare public metadata and file inventories against a target-compatible iPad3,1 firmware.
5. Keep P101AP/A6X kernel, device-tree, boot-chain, and firmware components marked **donor-only** unless a separate compatibility proof exists.
6. Use the donor primarily to study iOS 10 userspace layout, framework versions, resource formats, and build metadata.
7. Do not restore, flash, alter boot arguments, or overwrite the installed system as part of this research.

## Current conclusion

The iPad4 IPSW is a useful iOS 10 reference artifact, but it is not a drop-in base for an iPad3,1 IPSW. A genuine port would still require evidence that an A5X/J1AP-compatible kernel, device tree, boot chain, drivers, signing path, and userspace can work together. No such compatibility has been demonstrated.

No decryption, boot experiment, device-side change, or installation attempt was performed in this review.

## Change log

### 2026-11-17 — iPad4 donor IPSW assessed

- Recorded the supplied iPad3,4 / 14G60 / iOS 10.3.3 metadata and published SHA-256.
- Confirmed the donor/target product, board, and SoC distinction from public references.
- Classified the donor IPSW as analysis-only until component-level compatibility is demonstrated.
- Added a safe, non-destructive comparison workflow.

## Target IPSW baseline verified (2026-11-17)

A locally available iPad3,1 restore archive was inspected read-only. The archive manifest reports:

- Product type: `iPad3,1`
- Product version: iOS `9.3.5`
- Build: `13G36`
- Included kernel member: `kernelcache.release.j1`
- Archive SHA-256: `ceec5670fe63f113b766a09b189c50a31e96e4c538905076216fb21170385c95`
- Embedded kernelcache SHA-256: `232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e`

The embedded kernel member hash exactly matches the separately preserved kernelcache artifact already analyzed. The archive also contains J1AP-specific boot and device-tree components, including `DeviceTree.j1ap.img3`, `iBoot.j1.RELEASE.img3`, `LLB.j1.RELEASE.img3`, and S5L8945X display/recovery assets.

This confirms the current kernelcache provenance and gives us a clean target baseline for a future donor comparison. No archive contents were overwritten, no extraction output was created, and no device action was taken.

## Donor comparison status

The iPad3,4 / 14G60 donor archive is not present in the local workspace yet. Therefore, no claim is made about its actual component inventory beyond the public IPSW metadata. The next safe comparison requires obtaining that archive independently, verifying its published SHA-256, and inventorying it without restoring it.

## Donor acquisition check (2026-11-17)

The supplied IPSW Downloads page exposes an Apple download URL and publishes the expected SHA-256. A read-only fetch of that Apple URL returned HTTP 404 during this review. The donor archive was therefore **not** downloaded, and no unverified third-party mirror was substituted.

The published donor checksum remains:

```
3e2bd7d18823d93736a68d85961ff7ece5dd932a19c58d52e43282a6ea6ac312
```

The donor component comparison is pending until an independently obtained archive can be verified against that digest. This is an acquisition limitation, not evidence that the IPSW contents are compatible with iPad3,1.

## Verified donor archive and kernelcache comparison (2026-11-17)

The downloaded donor archive was inspected read-only:

- Product family: iPad4 Wi-Fi / `iPad3,4` (the archive also carries iPad3,5 and iPad3,6 identities)
- Product version: iOS 10.3.2
- Build: `14F89`
- Archive size: 1,930,214,085 bytes
- Archive SHA-256: `404689afbd30f4e418cc4a70aa7d18d4d5532fbafd6bce1314cbd1944230a26a`
- Kernel member: `kernelcache.release.ipad3b`
- Kernel member SHA-256: `0fb87e09cde7b5a0321ab34d982c0678b3fbe2429c96c38ef7cb0fab682922e3`

The donor kernelcache IMG3 parse is structurally complete:

| Logical tag | Offset | Total length | Data length |
|---|---:|---:|---:|
| TYPE | 0x14 | 0x20 | 0x04 |
| DATA | 0x34 | 0x00b1529c | 0x00b15282 |
| SEPO | 0x00b152d0 | 0x3c | 0x04 |

The final parsed offset is `0x00b1530c`, equal to the donor kernel member length. No KBAG tag is present in this donor kernelcache.

The donor DATA begins with the ASCII marker `complzss` at its payload start. Its header is followed by an LZSS-compressed stream whose apparent compressed stream begins at IMG3 file offset `0x1c0`; the first bytes there are `ff ce fa ed fe`. Public reverse-engineering references describe `complzss` as Apple's LZSS kernelcache wrapper and identify the decompressed 32-bit Mach-O marker as `ce fa ed fe`. This is strong evidence that this donor DATA is plaintext-compressed rather than protected by an IMG3 KBAG, but no decompression was performed in this step.

For comparison, the preserved target iPad3,1 kernelcache has:

- IMG3 tags TYPE, DATA, SEPO, KBAG, KBAG.
- DATA length 10,673,026 bytes.
- DATA first 16 bytes: `eef6429855e61ce16a030bee60bfe2e9`.
- DATA entropy: 7.999983875583 bits/byte.
- Two `aesType=0x100` KBAG entries.

This is a real container/format difference between the donor iOS 10.3.2 kernelcache and the target iOS 9.3.5 kernelcache. It does **not** establish that the donor kernel, device tree, boot chain, or drivers are compatible with J1AP/A5X, and it does not authorize a restore or boot attempt.

### Next safe analysis step

Use a source-auditable `complzss`/LZSS decoder on a separately written donor output, verify that the result begins with the expected 32-bit Mach-O marker, and record its Mach-O architecture and UUID. Preserve both IPSW archives and the original target kernelcache unchanged.

## Donor kernelcache LZSS decode (2026-11-17)

A source-matched in-memory LZSS decoder was applied to the donor's `complzss` stream. The implementation follows the public `lzssdec` algorithm and its documented Apple BootX provenance:

- [lzssdec source](https://github.com/malus-security/lzssdec/blob/main/lzssdec.cpp)
- [lzssdec documentation and Apple source reference](https://github.com/malus-security/lzssdec/blob/main/README.md)

No decoded output file was written.

Verified decode results:

| Measurement | Result |
|---|---:|
| Header-declared uncompressed size | 20,463,616 bytes |
| Compressed stream size | 11,620,610 bytes |
| Decoded size | 20,463,616 bytes |
| Decoded SHA-256 | `5e1a9e9fda3b7eb84efccb79cf0cf76979ea988270fcac667d8990db7b57ccc8` |
| Mach-O magic | `ce fa ed fe` |
| CPU type | `0x0000000c` (ARM) |
| CPU subtype | `0x0000000b` |
| File type | `0x00000002` (executable) |
| Load-command count | 19 |
| Load-command bytes | 3,108 |

The decoded kernel contains the public build string:

```
Darwin Kernel Version 16.6.0: Mon Apr 17 17:33:34 PDT 2017; root:xnu-3789.60.24~24/RELEASE_ARM_S5L8950X
```

This proves that the downloaded donor kernelcache can be decoded into a valid 32-bit ARM Mach-O executable. It does **not** prove that this kernel can run on the iPad3,1/A5X target. The CPU subtype, platform build, board/device-tree set, and driver configuration still require compatibility analysis. No boot or install attempt was made.

## Manifest-only experiment (2026-11-17)

A separate offline copy of the verified iPad4 10.3.2 IPSW was created. Its `BuildManifest.plist` was changed only to set `SupportedProductTypes = [iPad3,1]` and add a `PortExperiment` marker containing `NONINSTALLABLE=true`. A README was also added inside the archive.

This experiment intentionally leaves every donor payload unchanged: the A6X/ARMv7s kernelcache, P101AP/P102AP/P103AP device trees, iPad4 boot components, and signed firmware remain donor-specific. It is therefore **not a compatible or restorable IPSW** and must not be passed to Finder, iTunes, idevicerestore, or a boot tool.

The original downloaded archive and the verified original-copy archive were preserved separately. The manifest-only experiment is identified by SHA-256:

```
ba77aa2cb54f3a99e67393eb55d3c845650c5129465f93aadd71c2856c1dcfd6
```

### Compatibility plan after the routing experiment

1. Keep the manifest experiment as evidence only; do not test it on hardware.
2. Compare donor and target BuildManifest component roles and board/platform identifiers.
3. Compare decoded donor Mach-O architecture, CPU subtype, UUID, load commands, and platform strings with a legally obtained target-compatible kernel reference.
4. Analyze J1AP device-tree and boot-chain requirements separately from iOS 10 userspace.
5. Treat any component replacement as requiring signature/personalization analysis; do not assume a ZIP edit creates a bootable image.
6. Only consider a device-side experiment if static compatibility evidence and a recoverable test path exist.

## Mixed-component research bundle (2026-11-17)

Per the experiment choice, a second offline artifact was created from the verified iPad4 donor copy. It is explicitly labeled `NONINSTALLABLE` and contains a README warning against restore or boot use.

The following donor paths were replaced with byte-for-byte copies from the iPad3,1 IPSW:

- `DeviceTree.p101ap.img3` ← J1AP `DeviceTree.j1ap.img3`
- `iBoot.ipad3b.RELEASE.img3` ← J1 `iBoot.j1.RELEASE.img3`
- `LLB.ipad3b.RELEASE.img3` ← J1 `LLB.j1.RELEASE.img3`
- `iBEC.ipad3b.RELEASE.dfu` ← J1 `iBEC.j1.RELEASE.dfu`
- `iBSS.ipad3b.RELEASE.dfu` ← J1 `iBSS.j1.RELEASE.dfu`
- Apple-logo, battery, glyph, and recovery display images ← corresponding S5L8945X/J1AP images

The donor iOS 10.3.2 userspace, disk images, A6X/ARMv7s kernelcache, modem assets, and all unreviewed components were left unchanged. The manifest was marked for `iPad3,1` only and tagged `NONINSTALLABLE`.

Mixed-bundle SHA-256:

```
91994303694c367d2a7691ebdbfaa34e39dca591ede1e90c4549a112df62161e
```

The replacement entries were verified byte-for-byte against the target IPSW. This bundle is a controlled comparison artifact, not a compatibility result: its signatures, digests, kernel/platform pairing, and restore metadata are expected to be invalid or inconsistent. No device-side test was performed.

## Manifest-routing audit after the mixed bundle (2026-11-17)

The donor BuildManifest contains six build identities for device classes `p102ap`, `p103ap`, and `p101ap`, all pointing to the iPad4 kernelcache and iPad4 device-tree paths. The target BuildManifest contains two `j1ap` identities and only supports `iPad3,1`.

The mixed bundle changes its top-level `SupportedProductTypes` to `iPad3,1`, but intentionally retains the donor's six P101/P102/P103 identities. This is a deliberate warning: the mixed bundle is not even a coherent restore manifest, in addition to having invalid payload signatures and an A6X donor kernel. It must remain analysis-only.

No further manifest edits will be treated as a compatibility fix. A genuinely coherent port would require a new, target-specific manifest whose component digests, signing/personalization data, kernel, device tree, boot chain, and userspace all agree; the current evidence does not provide that.

## Device-tree and kernel identity audit (2026-11-17)

The donor and target device-tree IMG3 members differ materially:

| Artifact | Size | Logical tags | DATA entropy | DATA first bytes |
|---|---:|---|---:|---|
| Donor P101AP `DeviceTree.p101ap.img3` | 80,844 | TYPE, DATA, VERS, SEPO, CHIP, BORD | 3.00034442 | `13 00 00 00 0f 00 00 00 74 61 72 67 65 74 2d 74` |
| Target J1AP `DeviceTree.j1ap.img3` | 84,364 | TYPE, DATA, VERS, SEPO, CHIP, BORD, KBAG, KBAG | 7.99792937 | `1b ba b9 c0 11 76 81 2e a3 d6 ec ac 36 9a c9 ea` |

The donor device tree is a plaintext structured IMG3 DATA region; the target device tree is encrypted/keybagged in this artifact. This explains why the mixed bundle can carry the J1AP device-tree bytes as a research substitution but cannot be semantically compared without decryption, which remains out of scope.

The decoded donor iOS 10.3.2 kernel Mach-O has:

- UUID: `47ff35fed0c2392f9fed952adac7067e`
- ARM CPU subtype: `0x0b`
- File type: executable
- 19 load commands
- Segments including `__PRELINK_TEXT`, `__PRELINK_DATA`, and `__PRELINK_INFO`
- Decoded SHA-256: `5e1a9e9fda3b7eb84efccb79cf0cf76979ea988270fcac667d8990db7b57ccc8`

This gives us a stable donor kernel identity for future comparison. It does not establish that the A5X/J1AP target can execute an ARM subtype-`0x0b` kernel.

## Manifest hardware and digest comparison (2026-11-17)

The target-specific build identities expose a hard platform boundary:

| Field | Donor iPad4/P101AP | Target iPad3,1/J1AP |
|---|---|---|
| Device class | `p101ap` | `j1ap` |
| ApBoardID | `0x00` | `0x00` |
| ApChipID | `0x8955` | `0x8945` |
| ApSecurityDomain | `0x01` | `0x01` |

The donor and target manifests also carry different SHA-1 component digests:

| Component | Donor digest | Target digest |
|---|---|---|
| KernelCache | `47b753b58cfa09bdcd7f5d6fc251b4de42e3ef73` | `7aaedd449948798d4b054267f3ab0bc1d4762a3b` |
| DeviceTree | `ded32e0156187bf5d586544de95ce7bd80df0bbe` | `cbf2fa2ab97716a38ba192b76572037301fa9868` |
| iBoot | `793679755ec17f5a6560d7f0052e0ccf4703fd62` | `160460c4572a21171937d32c77dfaa2ce972d6fe` |
| LLB | `7932f7b1c4dee4f51ae2aa019f2d72de3fbb85b1` | `bb065b812b8a1079ee22e44c41fcdcb08ed97118` |
| iBEC | `7b52520541871388b68f0698303a6492ff773407` | `a35631d58a294df804fdbd9ada118984057360cb` |
| iBSS | `38a338c0766a13545f1424dc5006c4a69d40bfac` | `7273403dbd6150963f4ab917510a6a9f383da6a3` |

The differing `ApChipID` values are direct manifest evidence that the donor and target belong to different SoC families. The mixed bundle's component swaps do not resolve this boundary; its donor kernelcache, manifest digests, and signing/personalization relationships remain incompatible.

## Device-tree IMG3 metadata audit (2026-11-17)

The device-tree tags independently encode the same chip boundary as the manifests:

| Tag | Donor P101AP | Target J1AP |
|---|---:|---:|
| SEPO | 16 | 16 |
| CHIP | `0x8955` | `0x8945` |
| BORD | 0 | 0 |
| KBAG tags | none | two (`cryptState=1`, `cryptState=2`) |

The donor DATA begins with a structured binary header and has no KBAG tags. The target DATA begins with high-entropy bytes and is followed by two KBAGs. This is independent evidence that simply grafting J1AP device-tree bytes into the iPad4 donor does not produce a coherent iOS 10 image: the donor manifest, chip identity, encryption arrangement, and kernel remain from different platform generations.

## Boot-chain IMG3 metadata comparison (2026-11-17)

The chip mismatch is repeated across the boot chain, not limited to the kernelcache:

| Component family | Donor iPad4 | Target iPad3,1 |
|---|---|---|
| iBoot / iBEC CHIP | `0x8955` | `0x8945` |
| LLB / iBSS CHIP | `0x8955` | `0x8945` |
| SEPO | 16 | 16 |
| BORD tags | absent | values 0 and 1 |
| KBAG tags | absent | two AES-256-key-size entries |

The donor and target boot images are therefore signed/encrypted for different chip families and have different IMG3 metadata layouts. The mixed bundle's swapped J1AP boot files retain their target CHIP/KBAG metadata while the donor manifest and donor kernel remain P101AP/A6X-oriented. This is a concrete reason to keep the artifact offline-only; further blind substitutions cannot make it coherent.

## Donor kernel prelink inventory (2026-11-17)

The decoded donor Mach-O contains a `__PRELINK_INFO` region of 708,500 bytes. A byte-level scan found 180 distinct `CFBundleIdentifier` entries, including platform-sensitive families such as:

- `com.apple.driver.AppleARMPlatform`
- `com.apple.driver.AppleA5AE2`
- `com.apple.driver.AppleBCMWLAN*`
- `com.apple.driver.AppleH5GCLCD`
- `com.apple.driver.AppleMultitouchSPI*`
- `com.apple.driver.AppleNAND*`
- `com.apple.driver.AppleEmbeddedUSB*`
- `com.apple.driver.AppleInterruptController`

This is evidence that the donor kernelcache embeds a large, platform-specific prelinked driver set. The inventory alone cannot show whether the A5X/J1AP target's required implementations are ABI- or hardware-compatible; that requires a target-compatible decoded kernel/reference or source-level comparison. No kexts were extracted or modified.

## Digest interpretation correction (2026-11-17)

The `Digest` fields printed from Apple BuildManifest entries were compared with raw SHA-1 values of archive members. They do not equal the raw member SHA-1s, so this research does **not** interpret them as simple file hashes. They may represent Apple-specific payload/signature digest inputs.

The mixed bundle nevertheless retains the donor manifest's original digest metadata while replacing the corresponding component bytes with J1AP files. No digest recalculation, signing, or personalization was performed. The bundle remains non-installable and analysis-only.

## Userspace and restore-image inventory (2026-11-17)

The donor and target IPSWs contain different restore-image layouts:

| Item | iPad4 donor 10.3.2 / 14F89 | iPad3,1 target 9.3.5 / 13G36 |
|---|---|---|
| User system image | 1,862,226,470 bytes uncompressed | 1,615,699,968 bytes uncompressed |
| User ramdisk | 24,735,884 bytes uncompressed | 20,775,180 bytes uncompressed |
| Update ramdisk | 25,587,852 bytes uncompressed | 21,602,572 bytes uncompressed |
| Kernel mapping | platform ipad3b | target j1 |
| Ramdisk platform | s5l8950x | s5l8945x |
| Minimum system partition | 2367 | 2074 |

The donor's larger iOS 10 system image cannot be assumed to fit the target's iOS 9.3.5 partition layout. The donor and target also use different kernel/ramdisk platform keys and different restore-image mappings. This is a userspace/restore-layout compatibility concern in addition to the kernel and boot-chain mismatch.

No DMG was mounted, modified, or installed. The next useful userspace step is offline filesystem inventory of copied DMGs, if a suitable read-only DMG inspection tool is available.
## User system-image format audit (2026-11-17)

Read-only disk-image inspection added another compatibility boundary:

- Donor iOS 10.3.2 user image: UDZO, a compressed read-only disk image; hdiutil reported a GUID partition map and HFS+ partition metadata.
- Target iOS 9.3.5 user image: begins with the binary marker `encrcdsa`; hdiutil could not inspect it as a normal disk image without decryption.
- No image was mounted, decrypted, modified, or written back into either IPSW.

This means userspace filesystem comparison is currently asymmetric: the donor image can be structurally inspected, but the target image remains encrypted. A safe next step would require an authorized, reproducible decryption method and a separately written output; no such method has been used here.
## Public A5X/J1AP kernel-reference survey (2026-11-17)

A public-source survey found no iOS 10 restore IPSW or kernelcache identified for iPad3,1/J1AP. Public device references consistently list iPad3,1/A5X's latest compatible release as iOS 9.3.6, while iPad3,4/A6X has iOS 10.3.x releases:

- [The Apple Wiki: iPad3,1](https://theapplewiki.com/wiki/IPad3%2C1)
- [The Apple Wiki: iPad3,4/P101AP](https://theapplewiki.com/wiki/P101AP)
- [IPSW firmware comparison](https://www.ipsw.io/blog/ipad-3-4-firmware)

This is not a proof that a private or unreleased engineering kernel never existed. It does establish that the current public IPSW corpus does not supply the missing A5X/J1AP iOS 10 kernel reference. Milestone 17 remains blocked on that missing artifact or a source-level port.
## Remaining-requirements audit (2026-11-17)

### Source-level kernel path

Apple's public XNU repository provides source for the broader XNU family, and public release indexes identify XNU 3789.60.24 in the iOS 10 timeframe. That source is not the complete Apple iOS kernel build: platform drivers, proprietary headers, firmware interfaces, signing inputs, and device-specific configuration remain separate. A source-level A5X port would therefore require a new build configuration plus hardware-specific implementations and test evidence.

- [Apple XNU source repository](https://github.com/apple-oss-distributions/xnu)
- [XNU release index showing 3789.60.24](https://openhub.net/p/Darwin/commits)

### Signing and personalization

Apple's security documentation states that update authorization signs known cryptographic measurements and binds authorization to device values such as the ECID and anti-replay data. Editing an IPSW or replacing a component cannot recreate that authorization. The mixed bundles therefore remain offline research artifacts.

- [Apple Platform Security Guide](https://help.apple.com/pdf/security/en_GB/apple-platform-security-guide-b.pdf)
- [Apple secure software updates](https://support.apple.com/en-lamr/guide/security/secf683e0b36/web)

### Target filesystem boundary

The target user system image begins with encrcdsa and was not decrypted or mounted. Closing the filesystem comparison requires an authorized decrypted reference or a separately obtained filesystem snapshot. No encryption key, signing key, or bypass was requested or used.

### What remains genuinely required

1. A target-compatible A5X/J1AP iOS 10 kernel reference, or a complete source-level port with proprietary platform inputs.
2. A reproducible, authorized way to inspect the target filesystem image.
3. A legitimate signing/personalization path for any device-bound image.
4. A spare-device recovery plan and a validated bootable artifact before hardware testing.

## Public XNU source and build-feasibility audit (2026-11-17)

The official historical source URL for `xnu-3248.60.10` remains retrievable: it redirects from Apple Open Source to the tag in the Apple OSS Distributions repository. The downloaded source archive had SHA-256 `a4f646c6d34814df5a729a2c0b380c541dd5282b5d82e35e31bf66c034c2b761`.

A non-device build-feasibility check was performed in a disposable temporary directory. It did not read, modify, decrypt, or run either IPSW/kernelcache and produced no kernel artifact.

Confirmed source-tree limitations:

| Check | Result |
|---|---|
| Public source release | `xnu-3248.60.10` (iOS 9-era XNU series) |
| Public source declared supported architecture configurations | `X86_64`, `X86_64H` |
| J1AP / S5L8945X / ARMv7 target declaration | Not present in the public configuration searched |
| Embedded build entry point | Present as `make all_embedded` |
| Required default SDK | `macosx.internal` |
| Build result | Stops before target configuration because the internal SDK, associated tools, and embedded device map are unavailable |

The public build rules resolve SDK/tool paths using `xcrun` and request Apple-internal items such as `macosx.internal`, `embedded_device_map`, `migcom`, and CTF tools. The attempted `make all_embedded` invocation therefore fails with the missing internal SDK and then reports no target configurations. Separately, the source's `SUPPORTED_ARCH_CONFIGS` list contains only desktop x86 configurations, not ARMv7 or J1AP.

This is direct, reproducible evidence that the public XNU release is useful for source study but is not a complete, buildable A5X/J1AP iOS kernel source package. Constructing a real J1AP iOS 10 kernel would require the missing target build configuration, proprietary platform drivers and firmware interfaces, historical SDK/toolchain inputs, and an authorized signing/personalization path. None were substituted or bypassed.

References:

- [Apple XNU source tag: xnu-3248.60.10](https://github.com/apple-oss-distributions/xnu/tree/xnu-3248.60.10)
- [Apple XNU source tag: xnu-3789.60.24](https://github.com/apple-oss-distributions/xnu/tree/xnu-3789.60.24)
- [Current Apple OSS XNU repository](https://github.com/apple-oss-distributions/xnu)

## Change log

### 2026-11-17 — Public-source build feasibility checked

- Retrieved and hashed the official public XNU 3248.60.10 source archive in a disposable analysis directory.
- Inspected its declared architecture/configuration and embedded-build dependencies.
- Performed one non-device `all_embedded` build check; it stopped at the expected unavailable Apple-internal SDK/tooling boundary.
- Recorded that no target ARMv7/J1AP kernel was built and no device or firmware artifact was changed.

## Explicit iOS 10 ARM target check (2026-11-18)

The iOS 10-era public source was tested with an explicit target request:

```
make -j1 SDKROOT=/ HOST_SDKROOT=/ ARCH_CONFIGS=ARM KERNEL_CONFIGS=RELEASE MACHINE_CONFIGS=S5L8945X all
```

The build reached the kernel target-selection stage and stopped with:

```
Unsupported CURRENT_ARCH_CONFIG ARM
```

It also reported unavailable `migcom`, `ctfconvert`, and `ctfmerge` tools. This is stronger than a generic missing-SDK failure: even when an ARM/J1AP-like configuration is supplied manually, the public source rejects ARM as an unsupported architecture configuration before compilation. The source therefore cannot produce an iPad3,1 kernel by changing make variables alone.

The result was generated in temporary source storage only. No IPSW, kernelcache, signing data, or device was changed.

## Additional public analysis-tool survey (2026-11-18)

Several public tools can improve offline analysis, but none supplies the missing iOS 10 J1AP kernel:

- [ioskextdump_32](https://github.com/cocoahuke/ioskextdump_32) documents 32-bit kernelcache/kext inspection and includes an iOS 9-era sample workflow. It is relevant to analyzing a decoded ARMv7 kernel, not to creating one.
- [blacktop/ipsw device list](https://blacktop.github.io/ipsw/docs/guides/device_list/) records the target mapping as `iPad3,1 / j1ap / s5l8945x / armv7` and the donor mapping as `iPad3,4 / p101ap / s5l8955x / armv7s`.
- [ios-kexec-utils](https://github.com/axi0mX/ios-kexec-utils) documents ARMv7 tooling for already-jailbroken iOS 4.0–9.3.5 devices. It requires a jailbroken device and a kernel patch; it is not a safe substitute for a signed iOS 10 kernel and was not run.

These sources reinforce the existing conclusion: public tooling can inspect or load known ARMv7 artifacts under special conditions, but it does not provide the proprietary J1AP iOS 10 kernel, board configuration, drivers, or signing path. No device-side tool was executed.
