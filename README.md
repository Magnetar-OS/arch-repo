# entro314labs pacman repository

Prebuilt Arch Linux packages for entro314-labs desktop apps — the same
binaries published on each app's GitHub release, served as a real pacman
repository via GitHub Pages. Install and upgrade with plain `pacman`, no
AUR helper, no flatpak.

Packages are published here automatically by each app's release pipeline
([linux-release-kit](https://github.com/entro314-labs/linux-release-kit)'s
`arch-repo.yml`); nothing in this repo is edited by hand. The git history is
the repository's audit log.

## Use it

Add to `/etc/pacman.conf`:

```ini
[entro314labs]
SigLevel = Optional TrustAll
Server = https://magnetar-os.github.io/arch-repo/$arch
```

then:

```sh
sudo pacman -Syu
sudo pacman -S <app>
```

`x86_64` and `aarch64` are served; pacman picks yours via `$arch`.

### Verifying signatures (optional, recommended)

Packages and the repository database are GPG-signed. To make pacman enforce
that, import the key once and tighten `SigLevel`:

```sh
curl -sLo /tmp/entro314labs.asc https://magnetar-os.github.io/arch-repo/entro314labs.asc
sudo pacman-key --add /tmp/entro314labs.asc
sudo pacman-key --lsign-key "$(gpg --show-keys --with-colons /tmp/entro314labs.asc | awk -F: '/^fpr:/ {print $10; exit}')"
```

```ini
[entro314labs]
SigLevel = Required DatabaseRequired
Server = https://magnetar-os.github.io/arch-repo/$arch
```

## Layout

```
x86_64/            packages, detached .sig files, entro314labs.db + .files
aarch64/           same, for arm64
entro314labs.asc   the release signing public key
```

Superseded package versions are kept briefly for rollback
(`pacman -U x86_64/<older-file>.pkg.tar.zst`), then pruned; the database
always points at the newest.
