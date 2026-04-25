# Silicon Data Sleuthing

**Event:** HTB MCP TryOut · **Category:** forensics · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Sixteen-megabyte raw flash dump of an OpenWrt router. The container asks eight questions about the device configuration. All eight answers live inside the dump after you split it into its partitions. The trick is that the live `/etc/shadow` is not in the squashfs root, it is in the JFFS2 overlay, and you need `jefferson` to walk the overlay.

## What I saw first

`binwalk` reports a u-boot blob, a kernel, a squashfs, and what looks like another tarball at the tail. `binwalk` on this machine was broken because of a `capstone` library mismatch, so I did the partition split manually by scanning for magic signatures.

## What I tried that did not work

First instinct was to unsquashfs the rootfs and grep `/etc/shadow` for the root hash. The squashfs `/etc/shadow` was empty (no password set in the factory image). The user-set password lives somewhere else.

## What worked

OpenWrt's writable layer is a JFFS2 overlay at the tail of the flash. `jefferson` knows how to walk it.

```bash
# 1. Manual partition scan via magic signatures.
python3 -c '
data = open("dump.bin","rb").read()
print("squashfs at", data.find(b"hsqs"))   # 0x42c2c8
print("jffs2 at",    data.find(b"\\x85\\x19\\x03\\x20"))   # 0x7c0088 + overlay below
'

# 2. Extract the squashfs root.
unsquashfs -d sq -F 0x42c2c8 dump.bin

# 3. Extract the JFFS2 overlay.
dd if=dump.bin bs=1 skip=$((0x7c0000)) of=overlay.jffs2
pip install jefferson
jefferson -d overlay overlay.jffs2

# overlay/work/#<inode>/etc/shadow has the live root hash.
# overlay also contains the sysupgrade.tgz fragment with current /etc/config/* UCI files.
```

The eight answers map to:

| Question | File |
|---|---|
| OpenWRT version | `sq/etc/openwrt_release` |
| Linux kernel | `sq/usr/lib/opkg/info/kernel.control` |
| Root shadow line | `overlay/.../etc/shadow` |
| PPPoE username | `overlay/.../etc/config/network` |
| PPPoE password | `overlay/.../etc/config/network` |
| WiFi SSID | `overlay/.../etc/config/wireless` |
| WiFi password | `overlay/.../etc/config/wireless` |
| WAN→LAN redirect ports | `overlay/.../etc/config/firewall` |

## Flag

`HTB{Y0u'v3_m4st3r3d_0p3nWRT_d4t4_3xtr4ct10n!!_16af528ed6e040f7dfbd9a7ac48f49c1}`

## What this taught me

OpenWrt writes user changes to a JFFS2 overlay, not the squashfs root. `/etc/shadow` in the squashfs is the factory default. The live shadow is in the overlay. `jefferson` walks JFFS2 the way `unsquashfs` walks squashfs. If `binwalk` is broken (`capstone` version mismatch, the usual cause), you can scan for magic signatures manually with five lines of Python and skip the tool entirely.
