# Patterns

Short notes on tricks that worked in one place and are going to work in another. Each file is one pattern, named for the trigger that should make me reach for it.

The point of this folder is compounding. The first time I see a Unicode identifier in source code I lose twenty minutes. The second time I lose two minutes. After that I lose nothing, because the pattern is in here and I read it on the way in.

| Pattern | Trigger |
|---|---|
| [Invisible Unicode identifiers](invisible-unicode-identifiers.md) | Source code looks too clean to be exploitable |
| [Varnish cache key with no URL](varnish-cache-key-no-url.md) | Reverse-proxy with explicit `hash_data(req.http.X)` |
| [RAG/LLM as a file-read oracle](rag-llm-file-read-oracle.md) | Path traversal where content is consumed, not returned |
| [Two-stage supply-chain RCE](two-stage-supply-chain-rce.md) | Auto-update cronjob on a registry you can publish to |
| [Decode the jump table, not the prompt](decode-jump-table-not-prompt.md) | Movement keys that "obviously" map to compass directions |
| [Strings before Ghidra](strings-before-ghidra.md) | Rev challenge named like a search problem |
| [Rust per-byte closure array](rust-per-byte-closure-array.md) | Rev challenge with flag-length-many function pointers on the stack |
| [JFFS2 overlay on OpenWrt](openwrt-jffs2-overlay.md) | Router image where `/etc/shadow` in squashfs is empty |
| [EIP little-endian display vs memory](eip-little-endian-pattern-offset.md) | EIP shows ASCII chars but `--pattern o` returns -1 |
| [Hook fires at trigger, not at set](hook-fires-at-trigger-not-at-set.md) | A command or template stored now but evaluated later |
| [When the protocol is the moat, look for the side door](when-the-protocol-is-the-moat-look-for-the-side-door.md) | Primary service is a specialty or industrial protocol |
| [OPC-UA self-signed cert bypasses trust list](opcua-self-signed-cert-bypasses-trust-list.md) | OPC-UA endpoint advertising `Basic256Sha256` |
