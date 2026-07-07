<div align="center">

<h1>n0 void repo</h1>

<p>A cryptographically signed, self-updating XBPS package repository for Void Linux.</p>

<p><sup>Packages built via repoman · Signed & indexed automatically · Drop-in native xbps repo</sup></p>

</div>

---

## ⚡ Quick Setup

**① Add the repository**

```bash
echo 'repository=https://github.com/noeltz/n0_void-repo/releases/latest/download' \
  | sudo tee /etc/xbps.d/10-n0.conf
```

**② Sync and trust the signing key**

```bash
sudo xbps-install -S
```

> You'll be asked to import the RSA key for **`[n0] repoman build bot <actions@github.com>`** — press `y` to continue.

**③ Install anything**

```bash
sudo xbps-install <package-name>
```

---

## 🔄 Staying Updated

No extra steps — packages update with your system:

```bash
sudo xbps-install -Su
```

---

## 🔧 Development

This repository uses [repoman](https://github.com/noeltz/n0_void-repoman) to automate the package lifecycle.

### Secrets Required

| Secret | Description |
|---|---|
| `TOKEN` | GitHub PAT with `repo` scope |
| `XBPS_PRIVATE_KEY` | RSA private key for signing packages (PEM format) |

### Generate a Signing Key

```bash
openssl genrsa -out private.pem 4096
openssl rsa -in private.pem -pubout -out public.pem
```

Place `public.pem` in the repo root, and add `private.pem` contents as `XBPS_PRIVATE_KEY` secret.

### First Run

Trigger a manual workflow with **Force rebuild all packages** checked, since there are no cached binaries yet.

---

## 🛠 Troubleshooting

<details>
<summary><b>Repository not found</b></summary>
<br />
Verify <code>/etc/xbps.d/10-n0.conf</code> contains exactly:

```
repository=https://github.com/noeltz/n0_void-repo/releases/latest/download/
```
</details>

<details>
<summary><b>Key import failed</b></summary>
<br />
Place the public <code>public.pem</code> file manually into <code>/var/db/xbps/keys/</code>.
</details>

<details>
<summary><b>Package not found</b></summary>
<br />
Only <code>x86_64</code> glibc is currently supported.
</details>

---

<div align="center">

Powered by [repoman](https://github.com/noeltz/n0_void-repoman) · [Void Linux](https://voidlinux.org)

</div>
