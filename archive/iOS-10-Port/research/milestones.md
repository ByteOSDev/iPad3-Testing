# iPad 3 iOS 10 Port Milestone Ledger

Updated: 2026-11-17
Scope: Public, read-only research and offline artifacts. No device-side action.

Status meanings: Complete = reproduced and documented. Partial = meaningful evidence exists but the milestone is not fully closed. Blocked = requires unavailable evidence, authorization, or a safe recovery path.

## Milestones

| # | Milestone | Status | Finding / evidence |
|---:|---|---|---|
| 1 | Preserve iOS 9.3.5 installation and kernelcache | Complete | Original installation was not changed; preserved kernelcache SHA-256 is recorded in research/kernelcache.md. |
| 2 | Verify target IPSW checksum | Complete | iPad3,1 9.3.5 / 13G36 archive and embedded kernelcache were hashed and matched their recorded provenance. |
| 3 | Verify iPad4 donor IPSW checksum | Complete | iPad3,4 10.3.2 / 14F89 SHA-256: 404689afbd30f4e418cc4a70aa7d18d4d5532fbafd6bce1314cbd1944230a26a. |
| 4 | Inventory both IPSW archives | Complete | ZIP members, manifests, kernelcache, boot, device-tree, and DFU components were listed without altering either archive. |
| 5 | Parse both BuildManifest files | Complete | Product versions, supported product types, build identities, component paths, and manifest fields were inspected. |
| 6 | Record board/chip/security identifiers | Complete | Donor: P101AP / ApChipID 0x8955; target: J1AP / ApChipID 0x8945; both security domain 0x01. |
| 7 | Parse IMG3 structures | Complete | Kernelcache, device-tree, and boot-component tags were parsed with exact end offsets. |
| 8 | Compare kernelcache compression/encryption indicators | Complete | Donor DATA begins complzss and has no KBAG; target DATA is high-entropy and has two KBAGs. |
| 9 | Decode donor complzss kernelcache | Complete | In-memory LZSS decode produced 20,463,616 bytes beginning with ce fa ed fe. |
| 10 | Verify donor Mach-O metadata | Complete | ARM executable, subtype 0x0b, 19 load commands, UUID 47ff35fed0c2392f9fed952adac7067e. |
| 11 | Inventory donor prelinked drivers | Complete | 180 bundle identifiers were found in __PRELINK_INFO, including platform-sensitive driver families. |
| 12 | Compare boot-chain component roles | Complete | Donor and target have corresponding iBoot, LLB, iBEC, iBSS, and device-tree roles but different chip metadata. |
| 13 | Compare device-tree metadata | Complete | Donor CHIP 0x8955, no KBAG; target CHIP 0x8945, two KBAGs, and different DATA characteristics. |
| 14 | Compare manifest component metadata | Complete | Donor identities are P101/P102/P103; target identities are J1AP. Donor and target component metadata differ. |
| 15 | Build compatibility matrix | Complete | Kernel, device tree, boot chain, display, WLAN, NAND, USB, and driver differences are documented in research/ios10-base-compatibility.md. |
| 16 | Identify compatible iOS 10 userspace components | Partial | Donor system image is readable UDZO; target system image begins encrcdsa and remains uninspected without decryption. No runtime test has been performed. |
| 17 | Obtain an A5X/J1AP iOS 10 kernel reference | Blocked | No public iOS 10 J1AP/A5X IPSW or kernelcache was found. Official public XNU 3248.60.10 source declares only desktop x86 configurations and requires unavailable Apple-internal embedded SDK/device-map inputs; it cannot produce a J1AP kernel as released. |
| 18 | Resolve signing/personalization requirements | Partial | Manifest and IMG3 signing relationships are understood at a high level; no signing keys, personalization, or bypass has been attempted. |
| 19 | Create offline analysis bundles | Complete | Manifest-only and mixed-component bundles were created with explicit NONINSTALLABLE warnings. |
| 20 | Establish recovery gate for hardware testing | Blocked | No safe, validated bootable image or recovery procedure exists; no hardware experiment is authorized by the evidence. |

## Published offline artifacts

- Manifest-only experiment: SHA-256 ba77aa2cb54f3a99e67393eb55d3c845650c5129465f93aadd71c2856c1dcfd6.
- Mixed-component experiment: SHA-256 91994303694c367d2a7691ebdbfaa34e39dca591ede1e90c4549a112df62161e.
- Both artifacts are analysis-only and must not be restored or booted.

## Current conclusion

The static evidence demonstrates a real platform boundary: A5X/J1AP (0x8945) versus A6X/P101AP (0x8955), different boot-chain metadata, different device-tree encryption arrangements, and different prelinked kernel drivers. Further progress requires a target-compatible iOS 10 kernel/reference or source-level platform porting work. Blind archive edits cannot close these gaps.

No decryption, signing-key recovery, device-side modification, restore, or boot experiment was performed.

## Latest update — 2026-11-17

The donor/target restore-image and system-image audit is documented in research/ios10-base-compatibility.md. The target filesystem image is encrypted, so further filesystem comparison requires an authorized decryption method or a separately obtained decrypted reference. No device-side action occurred.

## Latest update — 2026-11-17 (remaining-requirements closure)

The remaining public, offline requirements were audited against the available evidence:

- **Source path:** Apple’s public XNU repository and the public XNU release index are now recorded as references. They do not provide the proprietary A5X/J1AP iOS 10 driver, firmware, signing, or device configuration inputs needed for a build.
- **Signing path:** Apple’s security documentation confirms that software authorization is based on signed measurements and device-bound values such as ECID/anti-replay data. An edited IPSW cannot recreate that authorization.
- **Filesystem path:** the target user image still begins with `encrcdsa`; no decryption or mount was attempted. A separately obtained, authorized decrypted reference remains necessary for a complete userspace comparison.
- **Public-kernel search:** no public iOS 10 J1AP/A5X kernel reference was found. Milestone 17 remains blocked; this is a missing-artifact limitation, not a compatibility claim.
- **Hardware gate:** no bootable artifact or recoverable test procedure exists, so no device-side testing is justified.

This closes the safe research/documentation work currently possible without decryption, signing-key recovery, a target-compatible kernel, or hardware experimentation.

## Latest update — 2026-11-17 (public source build check)

Milestone 17 was tested against Apple’s public historical XNU source rather than treated as an assumption. The official `xnu-3248.60.10` source archive was retrieved and hashed in a disposable directory. Its `all_embedded` build entry point requires `macosx.internal`, Apple-internal toolchain components, and an embedded device map; its declared supported architecture configurations are `X86_64` and `X86_64H`, not ARMv7/J1AP. The one build-feasibility invocation stopped before target selection, and no artifact was used or retained as firmware. This confirms that the public release cannot serve as a from-scratch A5X/J1AP iOS 10 kernel build kit.

## Latest update — 2026-11-18 (explicit ARM target check)

The iOS 10-era public XNU source was invoked with `ARCH_CONFIGS=ARM` and `MACHINE_CONFIGS=S5L8945X`. After the expected missing-tool warnings, the kernel make rules rejected the request as `Unsupported CURRENT_ARCH_CONFIG ARM`. This confirms that the public tree cannot be converted into an ARM/J1AP kernel through make-variable changes alone.

## Latest update — 2026-11-18 (public analysis tools)

The public-tool survey identified usable offline references for 32-bit kernelcache inspection and device mapping. These tools support analysis of decoded ARMv7 artifacts but do not provide a J1AP iOS 10 kernel or a signing path. No jailbreak, kexec, boot, or device-side tool was run.

## Latest update — 2026-11-18 (source coverage)

The public XNU 3789.60.24 tree was searched literally: it contains generic ARM code, but no S5L8945/S5L8950/S5L8955 identifiers and none of the donor kernel's platform-driver identifiers AppleARMPlatform, AppleA5AE2, AppleH5GCLCD, or AppleNAND. This confirms that the public source archive omits the hardware-specific driver source needed for a genuine J1AP/A5X port.