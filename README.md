# vPhone AIO

Virtual iPhone 17 Pro Max running iOS 26.3 on Apple Silicon Macs — an all-in-one launcher for iOS security research.

Forked from [34306/vphone-aio](https://github.com/34306/vphone-aio), based on [Lakr233/vphone-cli](https://github.com/Lakr233/vphone-cli).

---

## Requirements

| | |
|---|---|
| **Hardware** | Apple Silicon Mac (M1 / M2 / M3 / M4) — **not** Intel |
| **macOS** | Tahoe 26.x or Sequoia 15.x |
| **RAM** | 16GB minimum |
| **Disk** | 150GB+ free (12GB download, 69GB disk image, ~15GB extracted) |

---

## Installation

### Step 1 — Install Prerequisites

```bash
brew install git-lfs wget zstd libimobiledevice ldid
git lfs install
xcode-select --install
```

---

### Step 2 — Disable FileVault (Do This First)

FileVault must be **off** before entering Recovery, otherwise `csrutil authenticated-root disable` will fail.

```bash
sudo fdesetup disable

# Wait until fully decrypted before continuing
fdesetup status
# Must show: FileVault is Off.
```

> Decryption can take 30 min – 4 hrs depending on disk size. You can keep using the Mac, just don't force shutdown.

---

### Step 3 — Disable Security Layers (Recovery Mode)

Apple Silicon Macs have multiple security layers that all block vPhone. They must be disabled from **Recovery Mode**.

**Enter Recovery:**

1. Fully shut down the Mac
2. Hold the **power button** until you see *"Loading startup options..."*
3. Click **Options → Continue**
4. **Select your user and enter your password** ⚠️ *(critical — must authenticate here or nvram writes will fail)*
5. Open **Utilities → Terminal**

**Run all of these in Recovery Terminal:**

```bash
csrutil disable
csrutil authenticated-root disable
csrutil allow-research-guests enable
bputil -nkcas
nvram boot-args="amfi_get_out_of_my_way=1"
```

**Verify before rebooting:**

```bash
csrutil status
# System Integrity Protection status: disabled.

csrutil authenticated-root status
# Authenticated Root status: disabled.

nvram boot-args
# boot-args    amfi_get_out_of_my_way=1
```

**Reboot:**

```bash
reboot
```

> 💡 **If `nvram boot-args` returns `not permitted`**, use the GUID-prefixed form:
> ```bash
> nvram 7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args="amfi_get_out_of_my_way=1"
> ```

**Post-reboot check (normal macOS):**

| Command | Expected |
|---|---|
| `csrutil status` | `disabled.` |
| `csrutil authenticated-root status` | `disabled.` |
| `nvram boot-args` | `amfi_get_out_of_my_way=1` |
| `bputil --display-policy \| grep "Security Mode"` | `Permissive` |
| `fdesetup status` | `FileVault is Off.` |

---

### Step 4 — Clone This Repo

```bash
mkdir -p ~/viphone && cd ~/viphone
git clone --depth=1 https://github.com/dr34mhacks/vphone-aio.git
cd vphone-aio
git lfs pull # if not work go with the step 5 directly 
```

---

### Step 5 — Download Split Parts

The VM disk + tools ship as 7 split `.tar.zst` parts (~12GB total). The `-c` flag resumes interrupted downloads.

```bash
for p in aa ab ac ad ae af ag; do
  wget -c -O "vphone-cli.tar.zst.part_${p}" \
    "https://github.com/34306/vphone-aio/raw/refs/heads/main/vphone-cli.tar.zst.part_${p}?download="
done

# Verify all 7 parts downloaded (~1.7GB each)
ls -lh vphone-cli.tar.zst.part_*
```

---

### Step 6 — Extract & Build

```bash
chmod +x vphone-aio.sh
bash ./vphone-aio.sh
```

The script will:

```
[1/4] Check split parts
[2/4] Merge 7 parts into vphone-cli.tar.zst  (~12GB)
[3/4] Extract  (~15 minutes)
[3/4] Start iproxy tunnels
[4/4] Build & sign vphone-cli  (Swift build ~43s)
```

When it shows **"Connect via VNC / SSH"**, the build is complete. Press **Ctrl+C** to stop.

---

### Step 7 — Clean Up (Recover ~25GB)

```bash
cd ~/viphone/vphone-aio
rm -f vphone-cli.tar.zst.part_*
rm -f vphone-cli.tar.zst
rm -rf .git

# Confirm VM files exist
ls -lh vphone-cli/VM/
# AVPBooter.vresearch1.bin · AVPSEPBooter.vresearch1.bin · Disk.img (69GB)
# nvram.bin · SEPStorage · machineIdentifier.bin
```

---

## Important Notes

- **`csrutil allow-research-guests enable` is the critical step.** Without it, the VM starts but immediately throws `VZErrorDomain Code=1` with an empty serial log.
- This is a **research tool** — only run on a dedicated machine, never your daily driver. Disabling these protections removes macOS's core security model.
- The setup pulls binaries from the original [34306/vphone-aio](https://github.com/34306/vphone-aio) repo at install time.

---

## Credits

- [34306/vphone-aio](https://github.com/34306/vphone-aio) — all-in-one launcher
- [Lakr233/vphone-cli](https://github.com/Lakr233/vphone-cli) — upstream vphone-cli
- [wh1te4ever](https://github.com/wh1te4ever) — virtual iPhone research & writeup
