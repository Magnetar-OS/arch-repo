# Magnetar pacman repository

Prebuilt Arch Linux packages for [Magnetar](https://github.com/Magnetar-OS/magnetar)
— the distribution's own packages and the COSMIC application suite, served as a
real pacman repository. Install and upgrade with plain `pacman`, no AUR helper,
no flatpak.

Served from **`https://repo.magnetaros.com`**, a domain we own, rather than a
`github.io` address. That is deliberate: this URL ends up in every installed
machine's `/etc/pacman.conf`, and a `github.io` address encodes the GitHub
organisation name — which is exactly what broke when this repository moved from
`entro314-labs` to `Magnetar-OS`. A domain survives the next move.

Nothing here is edited by hand; the git history is the repository's audit log.

## Use it

Add to `/etc/pacman.conf`:

```ini
[magnetar]
SigLevel = Required DatabaseOptional
Server = https://repo.magnetaros.com/$arch
```

then:

```sh
sudo pacman -Syu
sudo pacman -S magnetar-desktop
```

`x86_64` and `aarch64` are served; pacman picks yours via `$arch`.

**On `SigLevel`.** `Required DatabaseOptional` is the right setting and the one
Magnetar ships. Import the signing key first, or pacman will correctly refuse
everything here:

```sh
curl -sLo /tmp/magnetar.asc https://repo.magnetaros.com/magnetar.asc
sudo pacman-key --add /tmp/magnetar.asc
sudo pacman-key --lsign-key "$(gpg --show-keys --with-colons /tmp/magnetar.asc | awk -F: '/^fpr:/ {print $10; exit}')"
```

Do not substitute `SigLevel = Optional TrustAll` to get past a signature error.
That setting means unsigned packages from this host run install scripts as root
without verification — acceptable while testing a repository you built
yourself, not acceptable on a machine you rely on.

## Layout

```
x86_64/          packages, detached .sig files, magnetar.db + .files
aarch64/         same, for arm64
magnetar.asc     the release signing public key
CNAME            repo.magnetaros.com
```

Superseded package versions are kept briefly for rollback
(`pacman -U x86_64/<older-file>.pkg.tar.zst`), then pruned; the database always
points at the newest.

## What is in here

One repository, not two. Distribution packages (`magnetar-repos`,
`magnetar-settings`, `magnetar-desktop`, `magnetar-calamares`) and the
application suite (`jump`, `peek`, `grabit`, `locket`, `envelope`, `circle`,
`slate`) are published together. They were split while the applications lived
in a different GitHub organisation; in one organisation that split is ceremony
with two publishing pipelines behind it.
