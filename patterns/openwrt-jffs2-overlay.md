# JFFS2 overlay on OpenWrt

**Trigger:** a router flash dump where `/etc/shadow` in the squashfs root is empty or contains only factory defaults.

OpenWrt's filesystem layout is read-only squashfs at the bottom plus a JFFS2 overlay at the top. Anything the user changes after first boot, including setting a root password, writing UCI configs, installing packages, lands in the overlay. The squashfs never changes.

So the shadow file in the squashfs is the empty factory copy. The shadow with the actual password hash lives in the JFFS2 overlay, somewhere near the tail of the flash image.

The tool that walks JFFS2 cleanly is `jefferson`. It is a `pip install` away and handles the node reconstruction the way `unsquashfs` handles squashfs.

```bash
# 1. Find the JFFS2 magic in the dump.
python3 -c '
data = open("dump.bin","rb").read()
print("jffs2 at", hex(data.find(b"\\x85\\x19\\x03\\x20")))
'

# 2. Extract just the JFFS2 region.
dd if=dump.bin bs=1 skip=$JFFS2_OFFSET of=overlay.jffs2

# 3. Walk it.
pip install jefferson
jefferson -d overlay overlay.jffs2

# The live /etc/shadow is in overlay/work/#<inode>/etc/shadow.
# The live /etc/config/* UCI files are alongside it.
```

If `binwalk` is broken (the usual reason is a `capstone` library version mismatch), a five-line Python script that scans for partition magic signatures gets you the offsets without it.

**Seen in:** [Silicon Data Sleuthing](../challenges/forensics/silicon-data-sleuthing.md).
