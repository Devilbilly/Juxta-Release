# Juxta releases

Download the executable for your platform, SHA256SUMS and KEY_ID from this repository's Releases page.
The signing key id is `1deec3d9d5dcbef8`; its public key is in `release.pub`.

Verify the download before installing:

```sh
sha256sum -c SHA256SUMS
chmod +x juxta-<version>-<platform>
./juxta-<version>-<platform> verify
./juxta-<version>-<platform> install --home "$HOME/juxta-home"
```

With an installed launcher, update with `juxta release install <downloaded-file> --home <home>`.
Compare KEY_ID with the signing key id above using a trusted copy of this repository.
The checksum detects download corruption; the executable also verifies its signed payload.
