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
| Preliminary IMG3 identification | Preliminary | Leading bytes consistent with IMG3 magic |
| Inspect first 256 bytes and decode header / initial tags | Complete | Header fields, TYPE and DATA tag headers decoded |
| Validate every IMG3 tag boundary | Pending | Need local parser and file-size consistency check |
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

## Immediate next step

Run a read-only Python parser on the **Mac** to enumerate all IMG3 tags and validate their lengths against the file size. The 256-byte header dump and SHA-256 fingerprint are already recorded in `research/kernelcache.md`.

## Following steps
1. Validate all IMG3 tag boundaries and file-size consistency.
2. Determine how the payload is stored and whether it is compressed.
3. Select a safe extraction tool or method for macOS.
4. Extract to a new file and verify its format.
5. Inspect Mach-O metadata.
6. Find a suitable iOS 10 32-bit comparison kernel and record its provenance.
7. Compare kernel and platform characteristics.
8. Reassess feasibility before considering any device-side experiment.

## Change log

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
