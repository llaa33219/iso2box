# 📦 iso2boxbuddy

**Turn any Linux ISO file into a ready-to-run Distrobox/BoxBuddy container automatically.**

> ✅ Lightweight alternative to virtual machines  
> ✅ Works on Bazzite, Fedora Atomic, Silverblue, Vanilla OS, and more  
> ✅ Supports almost all Linux distributions

---

## 🚀 Features

- Auto-detects and extracts squashfs from various ISO types:
  - `filesystem.squashfs`, `airootfs.sfs`, `rootfs.squashfs`, etc.
  - Special handling for NixOS (`squashfs-root`)
- Creates a working container with `distrobox` using the extracted rootfs
- Automatically recognized by **BoxBuddy GUI**
- Host folder sharing works out-of-the-box
- Requires no virtualization — just uses your host kernel

---

## 🖥️ Supported Platforms

- ✅ Bazzite OS
- ✅ Fedora Silverblue / Kinoite
- ✅ Vanilla OS / blendOS
- ✅ Any distro that supports `distrobox`

---

## 📦 Supported ISO Distributions

| Distro / ISO             | Supported | Method                         |
|-------------------------|-----------|--------------------------------|
| Ubuntu, Debian          | ✅        | `filesystem.squashfs`          |
| Arch, EndeavourOS       | ✅        | `airootfs.sfs`                 |
| Fedora, Rocky, Alma     | ✅        | `rootfs.squashfs`              |
| Kali, Parrot, Zorin     | ✅        | `live/filesystem.squashfs`     |
| NixOS                   | ✅        | `squashfs-root` (directory)    |
| Others with squashfs    | ✅        | Any `*.squashfs` or rootfs     |

---

## ⚙️ Quick Install

```bash
mkdir -p ~/bin && tee ~/bin/iso2boxbuddy.sh > /dev/null << 'EOF'
#!/usr/bin/env bash
# iso2boxbuddy - Universal ISO to Distrobox/BoxBuddy container converter
# Supports: Ubuntu, Debian, Fedora, Arch, NixOS, Kali, Parrot, Manjaro, etc.

set -e

if [ $# -ne 2 ]; then
    echo "Usage: $0 <path-to-iso> <container-name>"
    exit 1
fi

ISO_PATH="$(realpath "$1")"
CONTAINER_NAME="$2"
WORKDIR="$HOME/.local/share/iso2boxbuddy/$CONTAINER_NAME"
MOUNTDIR="/tmp/iso-mount-$CONTAINER_NAME"

# Dependency check
for cmd in unsquashfs distrobox mount find sudo cp; do
    if ! command -v "$cmd" &>/dev/null; then
        echo "❌ '$cmd' is not installed. Please install it first."
        exit 1
    fi
done

# Mount ISO
echo "📦 Mounting ISO: $ISO_PATH"
sudo mkdir -p "$MOUNTDIR"
sudo mount -o loop "$ISO_PATH" "$MOUNTDIR"

# Try to find root filesystem (standard squashfs or directory based)
SQUASH=$(find "$MOUNTDIR" \(
    -name "filesystem.squashfs" \
    -o -name "rootfs.squashfs" \
    -o -name "livecd.squashfs" \
    -o -name "airootfs.sfs" \
    -o -name "live/filesystem" \
    -o -name "*.squashfs" \
    -o -name "squashfs-root" \
    \) 2>/dev/null | head -n 1)

# Special case: NixOS squashfs-root as directory
if [ -z "$SQUASH" ]; then
    SQUASH=$(find "$MOUNTDIR/nix/store" -type d -name "squashfs-root" 2>/dev/null | head -n 1)
    if [ -n "$SQUASH" ]; then
        echo "📂 Detected NixOS squashfs-root directory. Copying contents..."
        mkdir -p "$WORKDIR/rootfs"
        cp -a "$SQUASH/." "$WORKDIR/rootfs/"
    fi
fi

# Extract if it's a squashfs file
if [ -n "$SQUASH" ] && [ -f "$SQUASH" ]; then
    echo "📂 Extracting squashfs file: $SQUASH"
    mkdir -p "$WORKDIR"
    unsquashfs -d "$WORKDIR/rootfs" "$SQUASH"
elif [ ! -d "$WORKDIR/rootfs" ]; then
    echo "❌ Could not find a valid root filesystem in ISO. Aborting."
    sudo umount "$MOUNTDIR"
    sudo rmdir "$MOUNTDIR"
    exit 1
fi

# Unmount and cleanup
echo "🔓 Unmounting ISO..."
sudo umount "$MOUNTDIR"
sudo rmdir "$MOUNTDIR"

# Create container
echo "🐳 Creating Distrobox container: $CONTAINER_NAME"
distrobox-create --name "$CONTAINER_NAME" --root "$WORKDIR/rootfs"

echo "✅ Success! You can now launch '$CONTAINER_NAME' from BoxBuddy."
exit 0
EOF

chmod +x ~/bin/iso2boxbuddy.sh
export PATH="$HOME/bin:$PATH"
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 🔧 Dependencies

Install these on OSTree-based distros like Bazzite:

```bash
rpm-ostree install squashfs-tools distrobox
```

> ⚠️ Reboot after installing if needed.

---

## ✅ Usage

```bash
iso2boxbuddy.sh /path/to/your.iso my-container-name
```

📦 Example:

```bash
iso2boxbuddy.sh ~/Downloads/ubuntu-22.04.iso ubuntu-from-iso
```

- Container will show up in **BoxBuddy** GUI automatically
- Host folders like Downloads, Documents, etc., will be accessible
- Perfect for testing distros or using dev environments

---

## 📁 File Structure

| Path                                | Purpose                            |
|-------------------------------------|------------------------------------|
| `~/.local/share/iso2boxbuddy/`     | Extracted ISO & container files    |
| `~/bin/iso2boxbuddy.sh`            | Your custom ISO-to-container tool  |
| BoxBuddy                           | GUI container manager integration  |

---

## 🛠 TODO / Future Features

- [ ] `.desktop` shortcut generation
- [ ] Preset ISO downloader (`--preset ubuntu`)
- [ ] GUI frontend (zenity or yad-based)
- [ ] Automatic AppImage container creation support

---

## 👤 Author

Made with ❤️ for the container community  
Contributions, pull requests, and bug reports welcome!

---

## 🪪 License

MIT — free to use, modify, and distribute.

