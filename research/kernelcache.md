# Kernelcache Acquisition and IMG3 Investigation

## Artifact source

The kernelcache was copied from the running iOS 9.3.5 installation at:

~~~text
/System/Library/Caches/com.apple.kernelcaches/kernelcache
~~~

Destination on the Mac:

~~~text
~/Desktop/iPad3-Research/kernelcache
~~~

## Acquisition command

Run on the **Mac**:

~~~sh
mkdir -p ~/Desktop/iPad3-Research

ssh -o HostKeyAlgorithms=+ssh-rsa root@10.0.0.142 \
  'cat /System/Library/Caches/com.apple.kernelcaches/kernelcache' \
  > ~/Desktop/iPad3-Research/kernelcache
~~~

The copy completed successfully. The resulting file is approximately 10 MB on the Mac; the source directory listing reports 11M.

## Initial file identification

Command on Mac:

~~~sh
file ~/Desktop/iPad3-Research/kernelcache
~~~

Output:

~~~text
.../iPad3-Research/kernelcache: data
~~~

The generic data result does not identify the inner payload.

## Initial header bytes

Command on Mac:

~~~sh
xxd -l 64 ~/Desktop/iPad3-Research/kernelcache
~~~

Output:

~~~text
00000000: 3367 6d49 8cdc a200 78dc a200 78dc a200  3gmI....x...x...
00000010: 6c6e 726b 4550 5954 2000 0000 0400 0000  lnrkEPYT .......
00000020: 6c6e 726b 0000 0000 0000 0000 0000 0000  lnrk............
00000030: 0000 0000 4154 4144 9cdb a200 82db a200  ....ATAD....
~~~

The first four bytes are 33 67 6d 49, ASCII 3gmI. This is the little-endian representation of the IMG3 magic value.

**Assessment:** IMG3 identification is confirmed by the magic, valid header fields, and complete tag parsing. The parser walked every tag to the exact end of the file.

## Strings inspection

Commands on Mac:

~~~sh
strings ~/Desktop/iPad3-Research/kernelcache | head -50
strings ~/Desktop/iPad3-Research/kernelcache | grep -Ei 'ARM_S5L|J1AP|AppleARM|Darwin Kernel' | head -30
~~~

The first command produced mostly unreadable output. The filtered command returned no matching strings. This is not enough to determine payload format or compression.

## 256-byte header and artifact fingerprint

Commands were run on the **Mac**:

~~~sh
xxd -s 0 -l 256 kernelcache
shasum -a 256 kernelcache
ls -lh kernelcache
~~~

Observed SHA-256:

~~~text
232caa3bfe34ff0ba841dd7c1e3a918aafd1b4376ead82f66f292c8d49d4021e
~~~

Observed size: approximately 10 MB (`ls -lh` reported `10M`).

The 256-byte dump confirms the following initial IMG3 structure (all integer fields are little-endian):

- Header magic bytes: `33 67 6d 49` (`3gmI`), corresponding to IMG3 magic.
- Header `fullSize`: `0x00a2dc8c`.
- Header `sizeNoPack`: `0x00a2dc78`.
- Header `sigCheckArea`: `0x00a2dc78`.
- Header identifier: `krnl` (kernel image).
- First tag: `TYPE`, with a 4-byte value identifying `krnl`.
- Next visible tag: `DATA`; its tag header reports total length `0x00a2db9c` and data length `0x00a2db82`.

These values are decoded from the supplied hex dump using the IMG3 header/tag layout. The subsequent parser pass validated every tag boundary and the final file-size relationship. The DATA bytes shown in the dump appear non-textual; that alone does not establish whether they are encrypted, compressed, or both.

## Full IMG3 tag parse

The user ran the parser on the **Mac**. It reported:

~~~text
TYPE     offset=0x00000014 total=0x00000020 data=0x00000004
DATA     offset=0x00000034 total=0x00a2db9c data=0x00a2db82
SEPO     offset=0x00a2dbd0 total=0x0000001c data=0x00000004
KBAG     offset=0x00a2dbec total=0x0000004c data=0x00000038
KBAG     offset=0x00a2dc38 total=0x00000054 data=0x00000038
End offset: 0xa2dc8c
File end:   0xa2dc8c
All bytes accounted for: True
~~~

**Confirmed:** the tag parser reached the exact end of the 10,673,292-byte file. No gaps, overlaps, or out-of-bounds tag lengths were reported. The complete observed tag sequence is `TYPE`, `DATA`, `SEPO`, `KBAG`, `KBAG`; no `SHSH` or `CERT` tags were reported by this parser.

`SEPO` is documented as the Security Epoch tag. Each `KBAG` has 56 bytes of tag data (`0x38`). The documented KBAG structure starts with two 32-bit fields (`cryptState` and `aesType`), followed by an encrypted IV and encrypted key. A 56-byte data area is consistent with the documented AES-256 keybag layout (8 bytes of fields + 16-byte IV + 32-byte key), but the two fields have not yet been read from this artifact. Do not claim AES-256 as confirmed until `aesType` is inspected. The existence of KBAG metadata indicates an encryption-related image format; it does not independently establish whether the DATA payload is additionally compressed.

## KBAG and SEPO field inspection

The user ran a read-only Python parser on the **Mac**. It decoded only the first eight data bytes of each KBAG and the four-byte SEPO value; it did not print the remaining encrypted IV/key material.

Observed output:

~~~text
SEPO security epoch: 16

KBAG at 0xa2dbec
  cryptState: 0x1
  aesType:    0x100

KBAG at 0xa2dc38
  cryptState: 0x2
  aesType:    0x100
~~~

**Confirmed from this artifact:** SEPO is 16. Both KBAG entries report `aesType=0x100`, which the documented IMG3 KBAG format defines as AES-256. The first entry reports `cryptState=0x1` (production GID-key state); the second reports `cryptState=0x2` (development GID-key state). This identifies the metadata fields and keybag types, not the actual key values. No key material was printed or recovered.

**Important limitation:** these metadata values do not decrypt the DATA payload. They also do not establish whether the DATA payload is compressed, or whether it uses an additional compression wrapper. No extraction or decryption has been performed.

## Not yet done
- Inspect the first 256 bytes. — Complete
- Record a SHA-256 digest. — Complete
- Parse and validate all IMG3 tag boundaries. — Complete
- Read SEPO and both KBAG metadata fields. — Complete
- Identify payload encoding and any compression format.
- Extract the kernel to a separate file.
- Verify whether the extracted object is a Mach-O ARMv7 kernel.
- Inspect Mach-O load commands and platform metadata.

## Next read-only step

Determine the DATA payload's storage/encoding and whether it has a compression wrapper. Choose a known, auditable Mac-side parser or extraction utility, first checking its documented behavior and ensuring it writes to a separate output file. Preserve the original kernelcache and do not attempt decryption or device-side changes as part of this step.

Do not overwrite the original kernelcache. Any extraction should write to a new path in ~/Desktop/iPad3-Research/.

## Interpretation cautions
- Do not infer payload compression solely from IMG3 magic or KBAG presence.
- The observed tag sequence is confirmed for this artifact; do not generalize it to every IMG3 image.
- Do not treat successful extraction as proof that the kernel is boot-compatible with another iOS release.
