# OPC-UA: a self-signed cert often walks straight through the trust list

**Trigger:** an OPC-UA endpoint (usually an "unknown" high TCP port that nmap can't fingerprint, sitting next to a Flask/SCADA web UI) that advertises a real security policy like `Basic256Sha256`.

OPC-UA looks intimidating from the outside. The endpoint announces signed-and-encrypted message modes, named security policies, X.509 client certificates. The instinct is to treat it like mutual TLS and assume you need a cert the server already trusts. You usually don't. A huge number of deployments ship with the trust list effectively open: the server will accept any client certificate as long as it is well-formed and the security policy matches. The cert is theater. It gets signed and encrypted, but never validated against a real CA or allowlist.

So the move is: generate your own self-signed cert and present it.

```
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/CN=client" -addext "subjectAltName=URI:urn:client:asyncua"
```

The `subjectAltName=URI:...` line matters. OPC-UA binds the application URI in the cert SAN to the client's `ApplicationUri`, and the handshake fails with a confusing error if they don't line up. Then connect with `asyncua`:

```python
from asyncua import Client, ua
from asyncua.crypto.security_policies import SecurityPolicyBasic256Sha256

client = Client("opc.tcp://TARGET:PORT")
await client.set_security(
    SecurityPolicyBasic256Sha256,
    certificate="cert.pem",      # kwarg is certificate=, not certificate_path=
    private_key="key.pem",
    mode=ua.MessageSecurityMode.SignAndEncrypt,
)
await client.connect()
```

That `certificate=` keyword trips people up because older snippets and other libraries use `certificate_path=`. Wrong kwarg, silent-ish failure.

Two things to do once you're in:

1. Pull the endpoints first with `get_server_endpoints()` (or `connect_and_get_server_endpoints()`) and read the policy and `UserIdentityTokens`. An empty token list means anonymous-with-cert is the intended path, which is exactly the gap above.
2. Walk the `objects` tree and filter Variables by their `AccessLevel` attribute. Only the nodes with the CurrentWrite bit set are worth poking; everything else is read-only noise.

And the payoff is frequently not on the OPC-UA server at all. See [[when-the-protocol-is-the-moat-look-for-the-side-door]]: the flag tends to surface on the sibling Flask + SocketIO dashboard that renders the live process state. Connect to that socket, subscribe to whatever update event the UI uses (`reactor_update` and friends), and scan every payload field for the flag. The protocol endpoint is how you change state. The web sidecar is how you read the result.
