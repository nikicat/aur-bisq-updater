# aur-bisq-updater

Daily GitHub Actions workflow that keeps the AUR package
[bisq](https://aur.archlinux.org/packages/bisq) (split packages:
`bisq-desktop`, `bisq-cli`, `bisq-daemon`) in sync with upstream
[bisq-network/bisq](https://github.com/bisq-network/bisq) releases.

## How it works

See [.github/workflows/update.yml](.github/workflows/update.yml):

1. Clones the AUR repo — **AUR is the single source of truth** for the
   PKGBUILD, so changes pushed directly by co-maintainers are picked up,
   never overwritten.
2. Compares `pkgver` against the latest upstream GitHub release.
3. On a new version: bumps `pkgver`/`pkgrel`, imports the pinned Bisq
   release signing key (fingerprint verified), and test-builds with
   `makepkg` — which verifies the PGP-signed release tag
   (`?signed` + `validpgpkeys` in the PKGBUILD) and checks out the
   `bitcoind` submodule revision recorded in that signed tag.
4. Regenerates `.SRCINFO`, commits `Update to version X`, pushes to AUR.

A failed run (bad signature, unexpected signing key, broken build) pushes
nothing and shows up as a failed Action.

## Setup

- Secret `AUR_SSH_PRIVATE_KEY`: a **dedicated** ed25519 key used only by
  this workflow; its public half is registered in the AUR account
  alongside the regular key, so it can be revoked independently.
- The `archlinux:base-devel` container image is pinned by digest (rolling
  distro — bump the digest now and then).
- Manual run: *Actions → Update AUR package → Run workflow*; with
  `force=true` it rebuilds the current version and dry-runs the push as an
  end-to-end test.
