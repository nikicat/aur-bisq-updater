# aur-bisq-updater

Daily GitHub Actions workflow that keeps the AUR packages
[bisq](https://aur.archlinux.org/packages/bisq) (split packages:
`bisq-desktop`, `bisq-cli`, `bisq-daemon`) and
[bisq2](https://aur.archlinux.org/packages/bisq2) in sync with upstream
[bisq-network/bisq](https://github.com/bisq-network/bisq) and
[bisq-network/bisq2](https://github.com/bisq-network/bisq2) releases.

## How it works

See [.github/workflows/update.yml](.github/workflows/update.yml); one
matrix job per package:

1. Clones the AUR repo — **AUR is the single source of truth** for the
   PKGBUILD, so changes pushed directly by co-maintainers are picked up,
   never overwritten.
2. Compares `pkgver` against the latest upstream GitHub release.
3. On a new version: bumps `pkgver`/`pkgrel`, imports the pinned Bisq
   release signing key (fingerprint verified), and test-builds with
   `makepkg -s` (installs whatever the PKGBUILD declares as dependencies).
   - `bisq`: makepkg verifies the PGP-signed release tag (`?signed` +
     `validpgpkeys` in the PKGBUILD) and checks out the `bitcoind`
     submodule revision recorded in that signed tag.
   - `bisq2`: upstream tags are lightweight and the commits are signed only
     by GitHub's web-flow key, so there is nothing to verify against a Bisq
     key. The PKGBUILD pins the git source by sha256 of `git archive`
     instead; the workflow refreshes that pin with `updpkgsums` on bump
     (trust on first use — it records what GitHub served at bump time).
4. Regenerates `.SRCINFO`, commits `Update to version X`, pushes to AUR.

A failed run (bad signature, checksum mismatch, broken build) pushes
nothing and shows up as a failed Action.

## Setup

- Secret `AUR_SSH_PRIVATE_KEY`: a **dedicated** ed25519 key used only by
  this workflow; its public half is registered in the AUR account
  alongside the regular key, so it can be revoked independently.
- The `archlinux:base-devel` container image is pinned by digest (rolling
  distro — bump the digest now and then).
- Manual run: *Actions → Update AUR packages → Run workflow*; with
  `force=true` it rebuilds the current version and dry-runs the push as an
  end-to-end test.
