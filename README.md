# 📦 iso2boxbuddy

**Turn any Linux ISO file into a ready-to-run Distrobox/BoxBuddy container automatically.**

> ✅ Lightweight alternative to virtual machines  
> ✅ Perfect for Bazzite, Fedora Atomic, Vanilla OS, and other container-focused Linux systems  
> ✅ Just one command to extract and launch from `.iso`

---

## 🚀 Features

- Extracts root filesystem from standard `.iso` files (via `filesystem.squashfs`)
- Creates a fully working Distrobox container from it
- Automatically compatible with **BoxBuddy GUI**
- Shares host folders (Downloads, Pictures, etc.) by default
- Much lighter and faster than traditional VMs (GNOME Boxes, VMware, etc.)
- Easy CLI usage: one-line install, one-line launch

---

## 🖥️ Supported Platforms

- ✅ Bazzite OS (Fedora Atomic-based)
- ✅ Fedora Silverblue / Kinoite
- ✅ Vanilla OS / blendOS
- ✅ Most Linux distros that support Distrobox

---

## ⚙️ Quick Install

```bash
mkdir -p ~/bin && tee ~/bin/iso2boxbuddy.sh > /dev/null << 'EOF'
#!/usr/bin/env bash
# iso2boxbuddy - Create a Distrobox container from a Linux ISO file

set -e

if [ $# -ne 2 ]; then
    echo "Usage: $0 <path-to-iso> <container-name>"
    exit 1
fi

ISO_PATH="$1"
CONTAINER_NAME="$2"
WORKDIR="$HOME/.local/share/iso2boxbuddy/$CONTAINER_NAME"

for cmd in 7z unsquashfs distrobox; do
    if ! command -v $cmd &>/dev/null; then
        echo "❌ '$cmd' is not installed. Please install it first."
        exit 1
    fi
done

echo "📦 Extracting ISO: $ISO_PATH"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
7z x "$ISO_PATH" -y

if [ ! -f "filesystem.squashfs" ]; then
    echo "❌ filesystem.squashfs not found. This ISO might not be supported."
    exit 1
fi

unsquashfs -d rootfs filesystem.squashfs

echo "🐳 Creating container: $CONTAINER_NAME"
distrobox-create --name "$CONTAINER_NAME" --root "$WORKDIR/rootfs"

echo "✅ Done! You can now open '$CONTAINER_NAME' from BoxBuddy."
EOF

chmod +x ~/bin/iso2boxbuddy.sh
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 🧩 Dependencies

```bash
rpm-ostree install p7zip squashfs-tools distrobox
```

> 🔁 Reboot your system after installing these if needed.

---

## ✅ Usage

```bash
iso2boxbuddy.sh /path/to/linux.iso my-container-name
```

📦 Example:

```bash
iso2boxbuddy.sh ~/Downloads/ubuntu-22.04.iso ubuntu-from-iso
```

- The container will appear in **BoxBuddy** automatically
- Host folders like `~/Downloads` will be accessible inside the container
- You can run GUI apps, CLI tools, or development environments

---

## 📁 File Structure

| Item                             | Description                                      |
|----------------------------------|--------------------------------------------------|
| `iso2boxbuddy.sh`                | Main auto-setup script                          |
| `~/.local/share/iso2boxbuddy/*` | Extracted ISO contents and rootfs per container |
| `BoxBuddy`                       | GUI frontend to manage the container easily     |

---

## 🛠 TODO

- [ ] `.desktop` launcher generation
- [ ] Preset ISO downloader (`--preset ubuntu`)
- [ ] GUI frontend for script (via Zenity)
