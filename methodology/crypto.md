# Crypto

The first thing I do on a crypto challenge is read the source. There is always source. The whole genre is "implement this incorrectly and ask the player to find the implementation bug". The implementation is the challenge.

## The first pass

Identify the primitive. The author named it something, but the name is decoration; the math tells you what it is. A function that takes a string and a key and XORs them is XOR regardless of what the author called it. A function that adds a per-position offset to each character is Trithemius (positional Caesar) regardless of what variable names suggest.

Once the primitive is named, the implementation flaw is usually one of:

- A weak RNG used to generate keys, IVs, or nonces.
- A constant or predictable IV reused across messages.
- A truncated or padded value where the truncation reveals enough bits to brute-force.
- A signature scheme implemented without proof-of-knowledge, allowing trivial public-key forgery.
- A custom protocol that does no work and ships plaintext.

## What I check before anything else

- The RNG. `random.randint` and `time.time` seeds are predictable. `os.urandom` is not, but it is often combined with predictable structures that leak entropy.
- Whether the key is constant across messages or varies. Stream ciphers with constant keys plus repeated plaintexts give the key away to crib-dragging.
- The padding. PKCS#7 implemented manually almost always has the oracle bug.
- The verification function. Many "signature" schemes verify by recomputing and comparing, which is fine, but the recomputation often skips a step (no `assert pk_is_valid`, no `assert sig_length == expected_length`) and the skip is the bug.

## What I avoid

Reaching for `sage` or `fpylll` before reading the source. Lattice attacks on broken RNGs are a real tool but a slow one. Read the source first. The bug is usually simpler than the heavy machinery would suggest.

Submitting the flag in lowercase. HTB and most platforms accept the literal case the source produces. Auto-normalizing eats half a submission per challenge for no reason.

## What I keep handy

- A short Python file with implementations of the named classical ciphers (Caesar, Vigenère, Trithemius, single-byte XOR, repeating-key XOR) so I do not re-derive them per challenge.
- `gmpy2` for big integers when modular arithmetic gets unwieldy.
- The `pycryptodome` library's `Crypto.PublicKey.RSA` for parsing keys and computing the math when the challenge is "factor this N".
- A note on which protocols have free oracles (CBC with a padding oracle, ECB with chosen-plaintext, RSA with no OAEP) so I do not waste time finding them again.
