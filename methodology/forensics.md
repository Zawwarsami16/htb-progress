# Forensics

Forensics challenges punish people who jump to tools. The right first move is always to figure out what the artifact actually is, before deciding which tool to point at it. A `.pcap` from a kerberoasting scenario and a `.pcap` from a USB-keyboard reconstruction scenario want completely different toolchains, and the only way to tell them apart is to look at the artifact first.

## The first pass

`file` and `binwalk -e` on the artifact. Always. Even when the file name says it is one thing, the magic bytes sometimes say it is another, and the wrapper around the real artifact is often a hint at the path the author wants me to take. If `binwalk` extracts a nested archive, recurse until the leaf.

For container formats — `.zip`, `.7z`, `.tar`, `.iso`, `.vhd`, `.E01` — mount or extract, then re-run `file` on the contents. Treat every layer as its own challenge until I find the actual signal.

## What I check by artifact type

- **`.pcap` / `.pcapng`** — open in Wireshark with `tcp.stream eq 0` and walk the streams. Filter on `http`, `dns`, `tls.handshake.extensions_server_name`, `usb.capdata`. Export objects (`File → Export Objects → HTTP/SMB/TFTP/IMF`) before doing anything bespoke; the flag is in an extracted file more often than in a packet.
- **Memory dump** — `vol3` with `windows.pslist`, `windows.cmdline`, `windows.malfind`, `windows.filescan`, then `windows.dumpfiles` on anything interesting. For Linux dumps, the right profile matters more than the right plugin; without the profile the dump is opaque.
- **Disk image** — `mmls` to see partitions, then `fls -r` per partition. `icat` to pull files by inode. The Windows registry hives (`SYSTEM`, `SOFTWARE`, `SAM`, `NTUSER.DAT`) carry persistence and credential signal that the filesystem alone does not.
- **USB keyboard pcap** — extract `usb.capdata`, decode the HID scancodes, watch for modifier keys. Most challenge authors leave the scancode-to-character table in the public domain; the trick is usually a shift-key hold around the flag braces.
- **Office documents** — `oletools` (`olevba`, `oleid`, `oledump`) for VBA macros, `oletools` again for embedded OLE objects. For `.docx`/`.xlsx`, unzip and grep the unzipped XML.
- **PDF** — `pdfid`, then `pdf-parser` to walk objects. Embedded JS lives in `/JavaScript` or `/JS` action objects. Embedded files live in `/EmbeddedFile`.
- **Image / steganography** — `exiftool` first, `binwalk` second, `zsteg` for PNG, `steghide` and `outguess` for JPEG with a passphrase guess. `stegsolve` for LSB walks on RGB planes.
- **Audio** — `audacity` spectrogram view. The flag is in the spectrogram more often than in any modulation scheme.

## What I avoid

- Reading hex dumps top to bottom looking for the flag. The flag is rarely literal, and when it is literal `strings | grep -i HTB{` finds it in two seconds.
- Running every Volatility plugin in sequence. `pslist` and `malfind` first, then I pick the next plugin based on what those two said.
- Mounting an unknown disk image read-write. Forensic artifacts are evidence; treat them as read-only or copy them first.

## When to stop and restart

If thirty minutes pass and I do not have a concrete next-action that follows from a concrete observation, the artifact identification was wrong. Go back to `file`, look harder at the magic bytes, search for `[unknown extension] CTF` to see whether the format has a standard parser I missed. Forensics rewards correct framing more than persistence.
