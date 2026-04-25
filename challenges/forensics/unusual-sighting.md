# An Unusual Sighting

**Event:** HTB Cyber Apocalypse 2024 (re-run in MCP TryOut) · **Category:** forensics · **Points:** 825 · **Difficulty:** very easy

## TL;DR

You have a bash history file and an SSH log. The container asks Q&A about the intrusion. Answer the eight forensic questions correctly and the container emits the flag. The flag is not in the raw evidence; it is what the container hands back after a correct walk-through.

## What I saw first

`bash_history.txt` and `sshd.log`. The log shows three normal logins from one IP and one anomalous login from a second IP using a different SSH pubkey fingerprint. The bash history shows the attacker running `whoami`, then a string of recon commands, ending with `./setup`.

## What I tried that did not work

Submitted four guessed flags built from the IOCs themselves. The attacker IP. The pubkey fingerprint. The URL of the malicious script. All rejected. The flag is not encoded into the evidence, even when the evidence looks like it should be the answer.

## What worked

Spawn the container and walk the Q&A. The eight answers I extracted from the local files are:

| Question | Answer |
|---|---|
| SSH server endpoint | `100.107.36.130:2221` |
| First successful login timestamp | `2024-02-13 11:29:50` |
| Unusual login timestamp | `2024-02-19 04:00:14` |
| Attacker source IP | `2.67.182.119` |
| Account used | `root` |
| Attacker SSH pubkey fingerprint | `OPkBSs6okUKraq8pYo4XwwBg55QSo210F09FCe1-yj4` |
| First attacker command | `whoami` |
| Last attacker command | `./setup` |

The container accepts the answers and emits the flag.

## Flag

`HTB{4n_unusual_s1ght1ng_1n_SSH_l0gs!}`

## What this taught me

For HTB challenges that ship both a static evidence file and a Docker Q&A service, the flag is the curated text the service emits after a correct walk. The evidence is there to let you derive the answers. Stop trying to guess the flag from the evidence directly. Spawn the container.
