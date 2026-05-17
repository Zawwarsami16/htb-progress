# Hardware

Hardware challenges in CTF context are almost never real hardware. They are a `.sal`, `.vcd`, `.lgw`, or `.csv` capture of bus traffic that the author wants me to reconstruct into characters, or a firmware image they want me to extract and search. The skill is reading the bus signal, not soldering an FPGA.

## The first pass

`file` on the artifact tells me the capture format. From there the right tool is fixed:

- `.sal` (Saleae v1+) → open in Logic 2. If Logic 2 cannot parse it, the file may be a `.sal` v0 wrapper around a different payload; unzip with `7z` and look at the raw doubles.
- `.vcd` → GTKWave, or grep the text directly for the signal of interest.
- `.lgw` → KingstVIS for Kingst LA captures; same idea as Logic 2.
- `.csv` of `(time, ch0, ch1, ch2, ...)` → load in Python or directly walk the columns with `awk`.

If the artifact is a firmware image (`.bin`, `.img`, `.elf`, `.uf2`, or an unknown binary blob with no header), it is a firmware-extraction challenge dressed as hardware. `binwalk -e` first, recurse on the extracted filesystem, then `file` on every leaf.

## What I check by bus type

- **UART** — single data line, idle high, start bit low. Decode by counting bit periods between transitions. The right baud is almost always 9600, 19200, 38400, 57600, or 115200; bruteforce all five if the author did not say. Frame is 8N1 in 95% of CTF cases. The flag is the ASCII string the line transmits.
- **SPI** — four lines: CLK, MOSI, MISO, CS. Sample MOSI on the rising edge of CLK while CS is low. Concatenate bytes. The flag is the MOSI byte stream, or it is the MISO byte stream when the slave is a flash chip being read back.
- **I2C** — two lines, SDA and SCL. Sample SDA on rising SCL. The start condition is SDA falling while SCL is high; the stop is SDA rising while SCL is high. Decode address and data bytes; identify the device from the first address.
- **JTAG** — four or five lines (TCK, TMS, TDI, TDO, optional TRST). Rare in CTF unless the author specifically wanted a JTAG-state-machine puzzle.

## What I check by firmware type

- ARM Cortex-M `.bin` — the first eight bytes are the initial stack pointer and reset vector. Cortex-M reset vectors point into the firmware itself, so the second word tells me the base address and reveals whether the binary is `0x08000000`-based (STM32) or `0x10000000`-based (RP2040) or something else.
- ESP32/ESP8266 firmware — `esptool.py image_info`, then split into bootloader, partition table, and application sections.
- OpenWrt / embedded Linux — `binwalk -e` extracts squashfs, jffs2, and ubifs. Mount with `unsquashfs` or `jefferson` and look at `/etc/shadow`, `/etc/config`, and the init scripts.

## What I avoid

- Trying to write a parser before checking whether the format has an existing one. Saleae and GTKWave are not optional; they are how everyone else solves these.
- Bruteforcing every possible UART config. Start with 8N1 at common baudrates; only widen if every common config produces garbage.
- Assuming a "hardware" challenge needs hardware. None of these have required a logic analyzer or scope on my end; the capture is the artifact.

## When to stop and restart

If the bus decode is producing garbage at every reasonable config, the channels are probably labelled wrong. Try the channels in a different order. If after three orderings every config still produces garbage, the format is probably not what `file` said it was; go back to step one.
