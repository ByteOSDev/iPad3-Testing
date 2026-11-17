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
