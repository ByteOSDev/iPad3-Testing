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

## Not yet done
- Inspect the first 256 bytes.
- Record a SHA-256 digest.
- Parse IMG3 header fields and tag boundaries.
- Identify payload and any compression format.
- Extract the kernel to a separate file.
- Verify whether the extracted object is a Mach-O ARMv7 kernel.
- Inspect Mach-O load commands and platform metadata.

## Next read-only commands

Run on the **Mac**:

~~~sh
xxd -s 0 -l 256 ~/Desktop/iPad3-Research/kernelcache
shasum -a 256 ~/Desktop/iPad3-Research/kernelcache
~~~

The hex dump gives more context for the container and tag layout. The SHA-256 value will serve as an artifact fingerprint. These commands do not contact or modify the iPad.

Do not overwrite the original kernelcache. Any extraction should write to a new path in ~/Desktop/iPad3-Research/.

## Interpretation cautions
- Do not infer payload compression solely from IMG3 magic.
- Do not assume a specific tag order or length until the header has been parsed.
- Do not treat successful extraction as proof that the kernel is boot-compatible with another iOS release.
