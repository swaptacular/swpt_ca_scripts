# Security and bug review: swpt_ca_scripts

| | |
|---|---|
| Reviewed commit | `fcd1e2a` (2026-03-20) |
| Review date | 2026-10-27 |
| Scope | the shell scripts (`init-ca`, `create-infobundle`, `sign-peercert`, `register-peer`, `sign-servercert`, `reconfigure-peer`, `generate-*key`, `peers/create-peer-manifest`) and `root-ca.conf.template` |
| Method | Manual review; the one fixed finding was confirmed by testing the validation predicate in isolation (no certificate forging) |

These scripts are a certificate authority operated by a node admin.
Their security job is to decide, from an **info-bundle supplied by
another party**, whether and how to sign a peer certificate. So the
untrusted input is the info-bundle passed to `sign-peercert`, and the
peer certificate passed to `register-peer`.

This document covers the issue **fixed in this pull request**, plus
review notes.

## Summary

| ID | Severity | Finding | Status |
|---|---|---|---|
| SEC-1 | Medium | Peer serial number used as a directory path after only an unanchored regexp check (path traversal) | Fixed |

## SEC-1: Path traversal through the peer serial number (Medium, fixed)

**Where:** `sign-peercert`, around line 220.

**Problem.** `sign-peercert` takes an info-bundle created by another
node, reads the `serialNumber` of the subject distinguished name out of
the enclosed root-CA CSR, and validates it with:

```sh
echo "$DN_SERIAL_NUMBER" | grep -E '[a-f0-9]+' > /dev/null || invalid_node_type
```

The regexp is **unanchored**, so it passes for any string that merely
*contains* one hex character. The value is then used directly as a
filesystem path:

```sh
peer_dir="$DIR/peers/$DN_SERIAL_NUMBER"
...
mv "$temp_dir" "$peer_dir"          # and, for an existing dir, rm -rf "$peer_dir/nodeinfo"
```

The `serialNumber` field is parsed with `get_dn_filed`, whose capture
is `[^,]*`, so it may contain `/` and `.`. A crafted info-bundle with a
subject such as `serialNumber = a/../../../../tmp/evil` therefore
passes the check and makes `sign-peercert` create directories and move
files **outside** the `peers/` directory.

For an *accounting authority* node the later `save_subnet` step calls
`match_public_key_hash`, which would reject a serial number that is not
the public-key hash — but that happens after `peer_dir` is already in
use, and it does not run at all for *creditors-agent* or
*debtors-agent* nodes. So the earliest check (this regexp) is the one
that has to be strict.

The attacker needs the node admin to run `sign-peercert` on the
attacker's info-bundle — but that is exactly the action the tool is for,
and every other check in the bundle (self-signed root cert, the
`nodeinfo` signature, `O = Swaptacular Nodes Registry`) is satisfied by
values the attacker controls.

**Confirmation (predicate only):**

```
current  'a/../../../../tmp/evil'  -> ACCEPTED
current  'abc; rm -rf x'           -> ACCEPTED
fixed    'a/../../../../tmp/evil'  -> rejected
fixed    '4ce2d4266d3b799c8df8ff7529450959' -> ACCEPTED   (a real serial)
```

**Fix.** Anchor the regexp to the whole string: `^[a-f0-9]+$`. This
matches the certificate spec, which defines node serial numbers as
lowercase hexadecimal, and rejects anything containing a path
separator or other character.

## Review notes

- **`register-peer`** also builds `peers/$DN_SERIAL_NUMBER` from the
  issuer of the received certificate, but requires that directory to
  already exist (and errors out otherwise). Since the directory can only
  have been created by `sign-peercert`, which now validates the serial
  number strictly, `register-peer` is covered. Anchoring its own
  extraction would be reasonable defence in depth.
- **Info-bundle unzip.** `sign-peercert` runs `unzip` on the untrusted
  bundle into a fresh `mktemp -d`. Info-ZIP's `unzip` drops `../`
  components and refuses absolute paths by default, and the script only
  reads fixed filenames from the result, so zip-slip is not a concern
  here.
- **DN parsing.** `get_dn_filed` uses `sed` with `([^,]*)`, so a DN
  value containing a comma would be truncated. For the fields used
  (`O`, `OU`, `serialNumber`) this is not exploitable, but a stricter
  parser (for example `openssl ... -nameopt`) would be more robust.
- **Duplicate-request protection.** `sign-peercert` refuses to sign a
  second, different root certificate for an already-known peer
  (comparing `root-ca.crt`), which correctly prevents re-issuing under a
  known serial number.
- **Private key handling.** `generate-masterkey` creates an
  AES-encrypted 3072-bit RSA key and refuses to overwrite an existing
  one. The CA config keeps `copy_extensions = none`, so a CSR cannot
  inject its own extensions.

## Tests

The repository has no automated test suite (these are operational shell
scripts). The fix was verified by testing the validation predicate on a
set of good and bad serial numbers, and with `bash -n`.
