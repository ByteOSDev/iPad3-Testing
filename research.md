# iPad 3 iOS 10 Port — General Research

**Repository:** https://github.com/ByteOSDev/iPad3-iOS10-Port  
**Status:** Early reconnaissance; no port attempt  
**Primary target:** Genuine iOS 10 on iPad 3,1 (J1AP)  
**Handoff date:** 2026-11-17

## 1. Project overview

This project investigates whether it is technically possible to port actual iOS 10 to an iPad 3,1. The objective is a genuine operating-system port, not an iOS 10-themed interface, SpringBoard customization, or Android recreation.

The approach is incremental, evidence-driven research involving firmware analysis, kernel and device-tree investigation, boot-chain compatibility, and userspace compatibility. AI-assisted reverse engineering may be used, but conclusions must be supported by artifacts or reproducible observations.

### Goals
- **Primary:** Investigate and, if feasible, run genuine iOS 10 on iPad 3,1.
- **Secondary:** Keep iOS 12 as a speculative research question only. Its 64-bit platform requirements make it unsuitable as a native target for this 32-bit device.
- Preserve the existing iOS 9.3.5 installation.
- Avoid unnecessary restores, updates, erasures, or device-side changes.
- Document discoveries, unresolved questions, and changes in understanding.

**Feasibility is unknown.** This repository does not claim a port is possible or that required drivers and boot support can be adapted.

## 2. Device summary

| Item | Finding | Confidence |
|---|---|---|
| Product identifier | iPad3,1 | Confirmed by device sysctl |
| Board identifier | J1AP | Confirmed by uname |
| SoC | Apple A5X | Platform identification |
| SoC identifier | S5L8945X | Confirmed in kernel build string |
| Architecture | ARMv7, 32-bit (arm) | Confirmed by kernel output |
| RAM | 1,034,944,512 bytes (~1 GB) | Confirmed by sysctl |
| Installed OS | iOS 9.3.5 | Confirmed by sw_vers |
| Build | 13G36 | Confirmed by sw_vers |
| Darwin | 15.6.0 | Confirmed by uname |
| XNU version | 3248.61.1 | Confirmed by uname |
| Kernel build | RELEASE_ARM_S5L8945X | Confirmed by uname |

See [research/hardware.md](research/hardware.md) for evidence and raw output.

## 3. Current device and development environment

The iPad currently runs iOS 9.3.5 with a checkm8-a5 jailbreak setup, normal boot, and working SSH/root-shell access. The device is reachable on the local network at 10.0.0.142; SSH username is root.

The SSH server only offers older host-key algorithms. This command successfully connected from the Mac:

~~~sh
ssh -o HostKeyAlgorithms=+ssh-rsa root@10.0.0.142
~~~

The development computer is an M1 MacBook Air. The working directory is:

~~~text
~/Desktop/iPad3-Research/
~~~

ChatGPT Work / Computer Use is unavailable in the user's workflow, so device and Mac commands must be supplied for the user to execute manually.

See [research/environment.md](research/environment.md).

## 4. Kernelcache artifact

The running iOS 9.3.5 kernelcache was copied from the iPad to the Mac:

~~~sh
mkdir -p ~/Desktop/iPad3-Research

ssh -o HostKeyAlgorithms=+ssh-rsa root@10.0.0.142 \
  'cat /System/Library/Caches/com.apple.kernelcaches/kernelcache' \
  > ~/Desktop/iPad3-Research/kernelcache
~~~

The copied file is approximately 10 MB on the Mac (the iPad directory listing reports 11M).

The file utility reports generic data. The first 64 bytes from xxd begin with:

~~~text
00000000: 3367 6d49 8cdc a200 78dc a200 78dc a200  3gmI....x...x...
00000010: 6c6e 726b 4550 5954 2000 0000 0400 0000  lnrkEPYT .......
00000020: 6c6e 726b 0000 0000 0000 0000 0000 0000  lnrk............
00000030: 0000 0000 4154 4144 9cdb a200 82db a200  ....ATAD....
~~~

The leading bytes 33 67 6d 49 correspond to ASCII 3gmI, the little-endian representation of the IMG3 magic. The IMG3 identification is confirmed by the header and full tag parse. The parser found `TYPE`, `DATA`, `SEPO`, and two `KBAG` tags, and reached the exact file end (`0xa2dc8c`) with all bytes accounted for. The SEPO value was read as `16`. Both KBAG entries report `aesType=0x100` (AES-256); their `cryptState` values are `0x1` (production GID-key state) and `0x2` (development GID-key state), respectively. These values identify the keybag metadata types, not the key material. The DATA payload has not been decrypted, decompressed, or extracted.

The 256-byte header dump and SHA-256 fingerprint have now been collected. The digest is `232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e`; the Mac reports a size of approximately 10 MB. The IMG3 header decodes to `fullSize=0x00a2dc8c`, `sizeNoPack=0x00a2dc78`, `sigCheckArea=0x00a2dc78`, and image identifier `krnl`. The first tags are `TYPE` (value `krnl`) and `DATA` (tag total length `0x00a2db9c`, data length `0x00a2db82`). Full tag-boundary validation is complete: all tags end exactly at the file boundary. The kernelcache has not been extracted or modified. See [research/kernelcache.md](research/kernelcache.md).

## 5. Compatibility research questions

The iPad 3 is a 32-bit ARMv7 device. iOS 10 supported some 32-bit devices, so ARMv7 alone does not settle the question. Support for other 32-bit devices does not establish that an iOS 10 build supports this exact A5X/J1AP platform or that its kernel and hardware support can be adapted.

Key unknowns:
1. Whether a suitable iOS 10 kernel and firmware baseline can be identified for comparison.
2. How the iOS 9.3.5 A5X kernel differs from iOS 10 kernels for supported 32-bit devices.
3. Whether kernel configuration, platform code, and device-tree expectations can be reconciled.
4. Whether the boot chain and firmware components can load a compatible kernel.
5. Whether display, touch, GPU, Wi-Fi, audio, storage, and other device drivers are available and compatible.
6. Whether iOS 10 userspace and frameworks can operate on this platform.
7. What signing, image-format, and firmware constraints apply.

iOS 12 is not considered a realistic native target because the iPad 3 is 32-bit and iOS 12 targets 64-bit devices. This is platform-level background reasoning, not the result of a device experiment.

See [research/compatibility.md](research/compatibility.md).

## 6. Confirmed findings vs. hypotheses

### Confirmed by observations
- The device identifies as iPad3,1 and J1AP.
- The running kernel reports RELEASE_ARM_S5L8945X and arm.
- The device reports 1,034,944,512 bytes of memory.
- The installed software reports iOS 9.3.5, build 13G36.
- SSH access to the normal booted installation works with the host-key option shown above.
- A kernelcache file was copied to the Mac.
- The copied file is a structurally validated IMG3 container; its full tag sequence is `TYPE`, `DATA`, `SEPO`, `KBAG`, `KBAG`, and the parser accounts for the complete file.

### Preliminary interpretation / hypotheses
- The kernelcache is wrapped in an IMG3 container. The header and complete tag boundaries have been validated.
- The payload may require an extraction/decompression step before it can be inspected as a Mach-O kernel. This has not been demonstrated.
- Porting may require substantial platform-specific kernel, device-tree, boot-chain, and driver work. The exact scope is unknown.

### Not established
- That an iOS 10 kernel can boot on the iPad 3.
- That any iOS 10 build contains required A5X/J1AP platform support.
- That the current kernelcache has been extracted.
- That a bootable port or compatible image exists.
- That an experimental boot has been attempted.

## 7. Preservation and research rules

- Keep the installed iOS 9.3.5 system intact.
- Prefer read-only inspection and local copies.
- Do not restore, update, erase, or overwrite the iPad as an exploratory step.
- Do not change NVRAM, boot arguments, bootloader components, kernelcache files, or system files without explaining consequences and obtaining explicit approval.
- Label commands clearly as Mac or iPad commands.
- Work in manageable batches and explain each command.
- Preserve exact outputs and distinguish observation from interpretation.
- Never treat an unverified extraction or compatibility assumption as completed work.

See [research/methodology.md](research/methodology.md).

## 8. Current status and immediate next step

The IMG3 container structure and metadata fields have been inspected. The confirmed values are SEPO `16`, and two AES-256 KBAG entries (`aesType=0x100`) with `cryptState` values `0x1` and `0x2` (production and development GID-key states, respectively). Only metadata fields were read; encrypted IV/key bytes were not printed. The DATA payload remains unextracted, undecrypted, and not yet classified for compression.

Next steps:
1. Determine the DATA payload's encoding and whether it is compressed.
2. Identify a safe, auditable Mac-side parser/extraction method.
3. If appropriate, extract to a new output file; never overwrite the original.
4. Determine whether the extracted data is a Mach-O ARMv7 kernel and inspect its metadata.
5. Obtain a legitimate iOS 10 kernelcache from a supported 32-bit device for comparison, where legally and practically available.
7. Compare kernel/platform information before making boot-compatibility claims.

See [research/status.md](research/status.md).

## 9. Future documentation automation

A future goal is an app or workflow that maintains this research repository. When a new discovery is made or an existing finding changes, the automation should update the relevant topic file and root research.md, preserve a dated change history, and avoid promoting unverified statements to confirmed findings.

**This automation does not exist yet and is not active.** At present, repository updates must be made explicitly during research sessions. A future implementation should include reviewable diffs, provenance for evidence, and safeguards against destructive device actions.
