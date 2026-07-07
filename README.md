```
          ____                 _     __
   ____  / __ \   _   ______  (_)___/ /     ________  ____  ____
  / __ \/ / / /  | | / / __ \/ / __  /_____/ ___/ _ \/ __ \/ __ \
 / / / / /_/ /   | |/ / /_/ / / /_/ /_____/ /  /  __/ /_/ / /_/ /
/_/ /_/\____/____|___/\____/_/\__,_/     /_/   \___/ .___/\____/
           /_____/                                /_/
```

a cryptographically signed, self-updating xbps binary repository for void linux.

packages are built via [repoman](https://github.com/noeltz/n0_void-repoman) and
published as github releases. add the repo, sync, and install.

---

## quick setup

```
$ echo 'repository=https://github.com/noeltz/n0_void-repo/releases/latest/download' \
    | sudo tee /etc/xbps.d/10-n0.conf

$ sudo xbps-install -S

  # trust the key for [n0] repoman build bot when prompted

$ sudo xbps-install vx
```

after that, packages update with your system (`xbps-install -Su`).

---

## available packages

| package | description | source |
|---|---|---|
| dgop | system/process monitor | github-release |
| openai-codex | openai codex cli | github-release |
| polo | local script manager | github-release |
| protonplus | wine/proton compatibility manager | github-release |
| vx | xbps convenience wrapper | github-release |
| zed | high-performance code editor | github-release |

---

## how to create a template

each package needs a directory under `srcpkgs/<pkgname>/` with a `template` file.
here are two examples to copy from:

### github-release (stable versions)

use this for projects that publish tagged releases on github.

```
pkgname=myapp
version=1.2.3
revision=1
archs="x86_64"
short_desc="One-line description of myapp"
maintainer="noeltz <https://github.com/noeltz>"
license="MIT"
homepage="https://github.com/owner/myapp"
distfiles="https://github.com/owner/myapp/archive/v${version}.tar.gz"
checksum=<sha256-of-distfile>

_n0_source=github-release
_n0_repo=owner/myapp
_n0_strip=v
```

repoman will:
- check the latest github release tag
- strip the `v` prefix to get the version
- download the distfile and compute the sha256 checksum
- bump `version`, reset `revision` to `1`, update `checksum`

get the initial checksum with:
```
$ curl -sL https://github.com/owner/myapp/archive/v1.2.3.tar.gz | sha256sum
```

### git-head (latest commit)

use this for projects where you want the absolute latest commit.

```
pkgname=myapp-git
_commit=0000000000000000000000000000000000000000
version=20260101000000
revision=1
build_style=cargo
short_desc="My app (git snapshot)"
maintainer="noeltz <https://github.com/noeltz>"
license="MIT"
homepage="https://github.com/owner/myapp"

_n0_source=git-head
_n0_repo="https://github.com/owner/myapp.git"
_n0_branch=HEAD

distfiles="https://github.com/owner/myapp/archive/${_commit}.tar.gz"
checksum=<sha256-of-distfile>
```

repoman will:
- run `git ls-remote` to get the latest commit hash
- set `_commit` to that hash
- set `version` to a utc timestamp (`yyyymmddhhmmss`)
- resolve `${_commit}` in the distfile url, fetch it, compute checksum
- reset `revision` to `1`

the `_n0_branch` field is optional — defaults to `HEAD`. to track a
specific branch, set it to the branch name (e.g., `develop`).

### template reference

| field | type | required | description |
|---|---|---|---|
| `pkgname` | string | yes | package name |
| `version` | string | yes | package version |
| `revision` | int | yes | xbps revision (repoman resets to `1`) |
| `archs` | string | no | target architectures (default: `x86_64`) |
| `short_desc` | string | yes | one-line description |
| `maintainer` | string | yes | who maintains this package |
| `license` | string | yes | spdx license identifier |
| `homepage` | string | yes | project url |
| `distfiles` | string | for update | source url (supports `${variable}`) |
| `checksum` | string | for update | sha256 of distfile |
| `_n0_source` | string | for update | `github-release` or `git-head` |
| `_n0_repo` | string | for update | `owner/repo` or full git url |
| `_n0_strip` | string | for github-release | tag prefix to strip (e.g., `v`) |
| `_n0_branch` | string | for git-head | branch to track (default: `HEAD`) |

---

## development setup

to run this repo locally you need:

- void linux (or the void-glibc-full container)
- `xbps-src` from [void-packages](https://github.com/void-linux/void-packages)
- [repoman](https://github.com/noeltz/n0_void-repoman)

### secrets

these must be set in the github repo:

| secret | description |
|---|---|
| `TOKEN` | github pat with `repo` scope |
| `XBPS_PRIVATE_KEY` | rsa private key (pem) for signing packages |

generate a keypair:

```
$ openssl genrsa -out private.pem 4096
$ openssl rsa -in private.pem -pubout -out public.pem
```

commit `public.pem` to the repo root. add the contents of `private.pem` as the
`XBPS_PRIVATE_KEY` secret in the repo settings.

### first run

trigger a manual workflow run with **force rebuild all packages** checked,
since there are no cached binaries yet.

---

## troubleshooting

<details>
<summary><b>repository not found</b></summary>

verify `/etc/xbps.d/10-n0.conf` contains exactly:

```
repository=https://github.com/noeltz/n0_void-repo/releases/latest/download/
```
</details>

<details>
<summary><b>key import failed</b></summary>

place `public.pem` manually into `/var/db/xbps/keys/`.
</details>

<details>
<summary><b>package not found</b></summary>

only `x86_64` glibc is currently supported.
</details>

---

*powered by [repoman](https://github.com/noeltz/n0_void-repoman) & [void linux](https://voidlinux.org)*

---

## credits

n0_void-repo is a fork of [abyss](https://github.com/clarajk/abyss) by
[clara keller](https://github.com/clarajk) — the original cryptographically
signed, self-updating xbps package repository for void linux. the repository
structure, workflow design, and automation approach are all directly inherited
from abyss.
