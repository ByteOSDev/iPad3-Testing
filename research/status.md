# Project Status and Next Steps

**Status date:** 2026-11-17  
**Overall phase:** Read-only reconnaissance  
**Experimental boot:** Not started

## Task checklist

| Task | Status | Notes |
|---|---|---|
| Identify iPad model | Complete | iPad3,1 from sysctl hw.machine |
| Confirm board identifier | Complete | J1AP from uname -a |
| Identify A5X / S5L8945X platform | Complete | Kernel build string and platform identification |
| Confirm 32-bit ARM architecture | Complete | uname -a reports arm |
| Collect RAM information | Complete | 1,034,944,512 bytes |
| Confirm installed iOS version | Complete | iOS 9.3.5, build 13G36 |
| Establish SSH access | Complete | Normal booted system; root shell |
| Locate kernelcache | Complete | iOS kernelcache directory |
| Download kernelcache to Mac | Complete | ~/Desktop/iPad3-Research/kernelcache |
| IMG3 container identification | Complete | Magic, header, and tags parsed |
| Validate every IMG3 tag boundary | Complete | All bytes accounted for; parser ended at exact file size |
| Record observed tag sequence | Complete | TYPE, DATA, SEPO, KBAG, KBAG |
| Inspect SEPO and KBAG metadata fields | Complete | SEPO=16; both aesType=0x100 (AES-256); cryptState values 0x1 and 0x2 |
| Inspect first 256 bytes and decode header / initial tags | Complete | Header fields and initial tags decoded |
| Record kernelcache SHA-256 | Complete | `232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e` |
| Determine payload encoding/compression | Pending | Not established |
| Extract kernel to separate file | Pending | Must not overwrite source |
| Verify extracted Mach-O format | Pending | Not started |
| Inspect Mach-O metadata/load commands | Pending | Not started |
| Obtain iOS 10 comparison kernel | Pending | Legitimate reference firmware needed |
| Compare iOS 9 and iOS 10 kernels | Pending | Not started |
| Investigate device-tree differences | Pending | Not started |
| Investigate boot-chain compatibility | Pending | Not started |
| Assess drivers and userspace | Pending | Not started |
| Experimental boot | Not started | No device-side modifications |

## Confirmed IMG3 metadata

- SEPO security epoch: `16`.
- First KBAG at `0xa2dbec`: `cryptState=0x1`, `aesType=0x100` (production GID-key state, AES-256).
- Second KBAG at `0xa2dc38`: `cryptState=0x2`, `aesType=0x100` (development GID-key state, AES-256).
- Only the metadata fields were read. Encrypted IV/key bytes were not printed or recovered.
- This does not establish DATA payload compression or decrypt the kernelcache.

## Immediate next step

Determine the DATA payload's encoding and whether it is compressed. Select a safe, auditable Mac-side inspection method before extracting anything. Preserve the original kernelcache and write any output to a separate file.

## Following steps
1. Identify the payload format and any compression wrapper.
2. Select and validate a local parser/extraction method.
3. If appropriate, extract to a separate output file.
4. Verify whether extracted data is a Mach-O ARMv7 kernel.
5. Inspect Mach-O metadata.
6. Find a suitable iOS 10 32-bit comparison kernel and record its provenance.
7. Compare kernel and platform characteristics.
8. Reassess feasibility before considering any device-side experiment.

## Change log

### 2026-11-17 — Format-focused review and reproducibility check
- Recomputed the local artifact SHA-256; it matches the recorded `232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e`.
- Measured exactly the declared DATA range: 10,673,026 bytes, remainder 2 modulo 16, 256 distinct byte values, and 7.999983875583 bits/byte Shannon entropy.
- Added source-linked IMG3/KBAG format references and clarified that `aesType=0x100` identifies the KBAG AES-256 key-size type, not a mode, key, or decryption result.
- Reclassified “production/development GID-key state” labels for `cryptState=0x1/0x2` as unsupported; retain the raw values only.
- No decryption, decompression, extraction, boot experiment, device-side action, or overwrite was performed.

### 2026-11-17 — KBAG and SEPO metadata inspection
- Read the four-byte SEPO value: `16`.
- Read the first two 32-bit fields of both KBAG entries without printing encrypted IV/key bytes.
- Both entries report `aesType=0x100` (AES-256); `cryptState=0x1` and `0x2` identify production and development GID-key states, respectively.
- No decryption, extraction, or device-side changes were performed.

### 2026-11-17 — IMG3 tag validation
- Parsed the complete IMG3 tag sequence: TYPE, DATA, SEPO, KBAG, KBAG.
- Confirmed every tag length is in bounds and the final offset exactly equals the 10,673,292-byte file size.
- Recorded two KBAG sections, each with 56 bytes of data; AES key size remains unconfirmed until the `aesType` fields are read.
- No extraction or device-side changes were made.

### 2026-11-17 — Kernelcache header inspection
- Inspected the first 256 bytes of the copied kernelcache on the Mac.
- Decoded IMG3 header fields and the initial TYPE (`krnl`) and DATA tag headers.
- Recorded SHA-256: `232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e`.
- Full tag-boundary validation, payload analysis, and extraction remain pending.
- No device-side changes were made.

### 2026-11-17 — Initial repository handoff
- Recorded iPad3,1 / J1AP / A5X / S5L8945X hardware findings.
- Recorded iOS 9.3.5 (13G36) environment and SSH access.
- Documented downloaded kernelcache and preliminary IMG3 identification.
- Recorded preservation requirements and outstanding analysis tasks.
- No kernel extraction, iOS 10 comparison, or boot experiment has been completed.


## DATA payload inspection — 2026-11-17

Read-only Mac-side analysis recorded:
- DATA payload size: 10,673,026 bytes.
- Unique byte values: 256 / 256.
- Shannon entropy: 8.0000 bits/byte.
- First 16 bytes: `eef6429855e61ce16a030bee60bfe2e9`.
- Raw signature counts: GZIP 146, XZ 0, BZIP2 2, LZ4 0, LZMA 0, ZLIB 164.

The first GZIP-like match at payload-relative offset `0x11d4c` is invalid: compression method `0xfa` (250) is not GZIP's required DEFLATE method 8, and flags `0xeb` set reserved bits. This is a false positive, not a verified GZIP stream. Other raw signature matches have not been structurally validated.

**Current assessment:** high entropy is consistent with encryption but does not prove it; compression has not been identified or excluded. The AES-256 KBAG metadata is confirmed, but the DATA payload remains undecrypted and unextracted.

No decompression, extraction, decryption, or device-side changes were performed. Do not attempt to decompress the invalid candidate.

### Next step
Identify a format-aware, auditable parser for this Apple kernelcache generation. Preserve the original file and direct any future extraction to a separate output path.
