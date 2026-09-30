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

**1. Trust the signing key.** `[magnetar]` is `SigLevel = Required`, so pacman
correctly refuses everything here until the key is known:

```sh
curl -sLo /tmp/magnetar.asc https://repo.magnetaros.com/magnetar.asc
sudo pacman-key --add /tmp/magnetar.asc
sudo pacman-key --lsign-key 382D984841F8C8A2BFB952675623FAF3DF36FFAF
```

Check the fingerprint before trusting it:

```
382D 9848 41F8 C8A2 BFB9  5267 5623 FAF3 DF36 FFAF
Magnetar OS (package signing) <packages@magnetaros.com>
rsa4096, created 2026-09-08, expires 2031-09-07
```

**2. Add the repository to `/etc/pacman.conf`, directly above `[cachyos]`**
(below the CachyOS optimised repositories such as `[cachyos-v3]`, if you have
them):

```ini
[magnetar]
SigLevel = Required DatabaseOptional
Server = https://repo.magnetaros.com/$arch
```

Position matters: pacman takes each package from the first repository that
carries it. This is the order a Magnetar install uses, and the one
`magnetar-repo-audit` checks — see
[REPOS.md](https://github.com/Magnetar-OS/magnetar/blob/main/docs/REPOS.md).

**3. Install.**

```sh
sudo pacman -Syu
sudo pacman -S magnetar-desktop      # x86_64: the whole Magnetar desktop
```

On **aarch64** only the applications are published — the distribution
packages (`magnetar-desktop` and the rest) are x86_64-only, as Magnetar is.
Install the apps you want by name, for example `sudo pacman -S jump envelope`.

`magnetar-desktop` pulls in `magnetar-keyring`, which ships this same key, so
`pacman-key --populate magnetar` keeps it current from then on; step 1 exists
for bootstrapping, where the keyring package cannot be verified yet because its
key is what you are installing.

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

The newest version of each package and the one before it are kept, for
rollback (`sudo pacman -U https://repo.magnetaros.com/x86_64/<older-file>.pkg.tar.zst`);
older ones are pruned when a new version is published. The database always
points at the newest. One superseded version, not more, because GitHub Pages
refuses to deploy a site over 1 GB, and the publishers fail before pushing a
tree that would exceed it.

## What is in here

One repository, not two: distribution packages and the application suite are
published together. They were split while the applications lived in a
different GitHub organisation; in one organisation that split is ceremony with
two publishing pipelines behind it.

| Kind | Packages | Architectures | Published by |
|---|---|---|---|
| Distribution | `magnetar-keyring`, `magnetar-repos`, `magnetar-settings`, `magnetar-branding`, `magnetar-desktop`, `magnetar-calamares`, `cutecosmic` | x86_64 | the Packages workflow in [Magnetar-OS/magnetar](https://github.com/Magnetar-OS/magnetar) |
| Applications | `jump`, `magnetar-peek`, `grabit`, `locket`, `envelope`, `circle`, `slate`, `magnetar-pencil`, `pocket` | x86_64, aarch64 | each app's release pipeline (linux-release-kit) |

Two applications carry the `magnetar-` prefix because their plain name was
taken: the previewer is `magnetar-peek`, since Arch already has an unrelated
`peek` (a GIF recorder), and the editor is `magnetar-pencil`, since other
repositories carry an unrelated `pencil` (Evolus Pencil). Each replaces the
package this repository used to publish under the plain name (`peek` ≤ 1.0.1,
`pencil` ≤ 1.2.0) when you upgrade, and nothing else by that name.

A package that another one here replaces does not stay: the Packages workflow
removes the old name from the databases and from both directories, so it can
no longer be installed from here by that name.
