# Phreaky

**Event:** HTB MCP TryOut · **Category:** forensics · **Points:** 900 · **Difficulty:** very easy

## TL;DR

PCAP of SMTP traffic. Fifteen emails, each carrying a password-protected zip with one part of a PDF. The per-zip password is sitting in the email body. Reassemble parts one through fifteen into a PDF, extract the body, find the flag in the last line.

## What I saw first

`phreaky.pcap` inside a zip with password `hackthebox` (HTB's default for challenge archives). The capture is straightforward SMTP from `caleb@thephreaks.com` to `resources@thetalents.com`.

## What I tried that did not work

Tried John on the zip passwords first out of habit. Useless when the password is in plaintext two centimeters above the attachment.

## What worked

```bash
unzip -P hackthebox phreaky.zip
tshark -r phreaky.pcap --export-objects imf,mails -q

# Each .eml in mails/ has:
#   - text/plain body containing "Password: <12 chars>"
#   - base64 application/zip attachment
#
# Parse each .eml, extract password from body, decrypt the zip with it,
# pull phreaks_plan.pdf.partN out.
python3 << 'EOF'
import email, base64, re, zipfile, io
for i in range(1, 16):
    m = email.message_from_file(open(f"mails/mail-{i}.eml"))
    pw = re.search(r"Password:\s*(\S+)", m.get_payload(0).get_payload()).group(1)
    z  = io.BytesIO(base64.b64decode(m.get_payload(1).get_payload()))
    with zipfile.ZipFile(z) as zf:
        zf.extractall("parts", pwd=pw.encode())

# Concatenate part 1..15 in order.
with open("plan.pdf","wb") as o:
    for i in range(1,16):
        o.write(open(f"parts/phreaks_plan.pdf.part{i}","rb").read())
EOF

pdftotext plan.pdf -
# ...
# HTB{Th3Phr3aksReadyT0Att4ck}
```

## Flag

`HTB{Th3Phr3aksReadyT0Att4ck}`

## What this taught me

HTB challenge zips have a default password of `hackthebox`. Try that first before anything else. `tshark --export-objects imf` is the fastest way to extract MIME emails from a pcap. And when a multi-part zip series has password-protected attachments, the passwords are always inline in the email body. The regex `Password:\s*(\S+)` covers ninety-nine percent of the variants.
