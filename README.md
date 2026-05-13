# deb-ikafssn

A standalone APT channel for [ikafssn](https://github.com/astanabe/ikafssn).

This repository hosts the GPG-signed APT channel served from
`https://deb.ikafssn.org` (GitHub Pages with the `deb.ikafssn.org`
custom domain). Unlike the Conda channel, the `.deb` packages
themselves are stored *in this repository* under `pool/`, because APT's
`Filename:` field is required to be a relative path inside the channel
and GitHub Pages cannot serve real HTTP redirects.

The channel always reflects the latest ikafssn release; older versions
are not retained. Past versions can still be downloaded directly from
[ikafssn's GitHub Releases](https://github.com/astanabe/ikafssn/releases).

## Installation

One-time setup (run as root or via `sudo`):

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://deb.ikafssn.org/ikafssn-archive-keyring.asc \
  | sudo gpg --dearmor -o /etc/apt/keyrings/ikafssn-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/ikafssn-archive-keyring.gpg] https://deb.ikafssn.org/ $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/ikafssn.list
sudo apt update
sudo apt install ikafssn
```

Subsequent upgrades:

```bash
sudo apt update && sudo apt upgrade ikafssn
```

## Supported suites

| Suite | Ubuntu release | Architectures |
|---|---|---|
| `jammy` | Ubuntu 22.04 LTS | `amd64`, `arm64` |
| `noble` | Ubuntu 24.04 LTS | `amd64`, `arm64` |

## Layout

```
CNAME, README.md, ikafssn-archive-keyring.asc       (static; one-time)
pool/main/i/ikafssn/*.deb                            (regenerated per release)
dists/<suite>/Release, Release.gpg, InRelease        (regenerated per release)
dists/<suite>/main/binary-<arch>/Packages{,.gz}      (regenerated per release)
```

## How this channel works

`apt-get update` triggers:

1. APT fetches `https://deb.ikafssn.org/dists/<suite>/InRelease`,
   which is GPG-clearsigned by ikafssn's signing key (verified against
   the public key dearmored into `/etc/apt/keyrings/`).
2. APT then fetches `Packages` for the appropriate architecture,
   verifying the hashes listed in `InRelease`.
3. `apt-get install ikafssn` resolves the entry in `Packages`, follows
   its `Filename:` (a relative path under `pool/`), and downloads the
   `.deb` from this same channel host.

## Release flow

When a new ikafssn release is published, the channel layout under
`pool/` and `dists/` is regenerated from scratch. This is automated by
ikafssn's
[`release.yml`](https://github.com/astanabe/ikafssn/blob/main/.github/workflows/release.yml)
workflow (`update-deb-channel` job): after all `.deb` artifacts are
uploaded to the release, the helper `recipe/deb-publish.sh`
- wipes `pool/` and `dists/`,
- downloads the four `.deb` assets,
- runs `dpkg-scanpackages` and `apt-ftparchive` per suite,
- signs each `Release` with `gpg --clearsign` (→ `InRelease`) and
  `gpg --detach-sign --armor` (→ `Release.gpg`),

and pushes the result to `main` here.

## Verifying the public key

The same `ikafssn-archive-keyring.asc` is committed at the root of
the [astanabe/ikafssn](https://github.com/astanabe/ikafssn)
repository, so the key can be fetched out-of-band and compared against
the one served by this channel before trusting it.

## License

Apache-2.0 (matching ikafssn).
