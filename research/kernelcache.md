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

**Assessment:** IMG3 is a strong preliminary container identification based on the leading magic. The container header and tag records still need structural validation.

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

These values are decoded from the supplied hex dump using the IMG3 header/tag layout. A complete parser pass is still required to validate every tag boundary and the final file-size relationship. The DATA bytes shown in the dump appear non-textual; that alone does not establish whether they are encrypted, compressed, or both.

## Not yet done
- Inspect the first 256 bytes. — Complete
- Record a SHA-256 digest. — Complete
- Parse IMG3 header fields and tag boundaries. — Pending structural parser validation.
- Identify payload and any compression format.
- Extract the kernel to a separate file.
- Verify whether the extracted object is a Mach-O ARMv7 kernel.
- Inspect Mach-O load commands and platform metadata.

## Next read-only step

Run a local parser on the Mac to enumerate and validate every IMG3 tag boundary. This can be done with Python using only the local file; it does not contact or modify the iPad. Do not extract or alter the source file until the parser output has been checked.

Do not overwrite the original kernelcache. Any extraction should write to a new path in ~/Desktop/iPad3-Research/.

## Interpretation cautions
- Do not infer payload compression solely from IMG3 magic.
- Do not assume a specific tag order or length until the header has been parsed.
- Do not treat successful extraction as proof that the kernel is boot-compatible with another iOS release.
