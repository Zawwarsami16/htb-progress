# RAG/LLM as a file-read oracle

**Trigger:** path traversal where the file content is consumed by a downstream model or search index, rather than returned to the user directly.

A common pattern in LLM-integrated apps:

1. An endpoint takes a "topic" or "document name", builds a path with `f"{baseDir}/{topic}{suffix}"`, and reads the file content.
2. The content is fed into a RAG store, a vector index, or a Q&A model.
3. A second endpoint lets the user ask questions whose answers may include text from the stored documents.

If step one has a traversal bug, the user can plant arbitrary file content into the model's accessible context. Step three becomes a file-read oracle. Ask the model a direct question about the file content and it will recite it.

The model does not need to be jailbroken. It is doing its job. The bug is in the upstream loader that let an attacker stuff `/etc/passwd` or `/app/config.py` into the documents the model is supposed to draw from.

Defense is the same as any traversal bug: canonicalize the path, reject anything outside the intended directory, refuse `..` segments. The model layer is downstream and cannot fix this.

**Seen in:** [Chrono Mind](../challenges/misc/chrono-mind.md).
