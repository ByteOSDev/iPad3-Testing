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
| Inspect complete IMG3 metadata | Pending | Need larger header dump and parser |
| Record kernelcache SHA-256 | Pending | Suggested command not yet run |
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

Run on the **Mac**:

~~~sh
xxd -s 0 -l 256 ~/Desktop/iPad3-Research/kernelcache
shasum -a 256 ~/Desktop/iPad3-Research/kernelcache
~~~

Share both outputs for analysis. These commands are read-only and do not modify the iPad.

## Following steps
1. Validate IMG3 header and tag layout.
2. Determine how the payload is stored and whether it is compressed.
3. Select a safe extraction tool or method for macOS.
4. Extract to a new file and verify its format.
5. Inspect Mach-O metadata.
6. Find a suitable iOS 10 32-bit comparison kernel and record its provenance.
7. Compare kernel and platform characteristics.
8. Reassess feasibility before considering any device-side experiment.

## Change log

### 2026-11-17 — Initial repository handoff
- Recorded iPad3,1 / J1AP / A5X / S5L8945X hardware findings.
- Recorded iOS 9.3.5 (13G36) environment and SSH access.
- Documented downloaded kernelcache and preliminary IMG3 identification.
- Recorded preservation requirements and outstanding analysis tasks.
- No kernel extraction, iOS 10 comparison, or boot experiment has been completed.
