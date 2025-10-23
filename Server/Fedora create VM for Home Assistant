Here’s a well-formatted **`README.md`** version of your instructions with clear explanations for each step:

---

# 🏠 Home Assistant Installation via KVM/QEMU (Libvirt)

This guide explains how to install **Home Assistant OS** on a Linux system using **KVM/QEMU (libvirt)**.
We’ll download the official Home Assistant QCOW2 image, remove any existing VM, and create a new one with a direct network bridge.

---

## 📦 Prerequisites

Ensure you have the following installed on your system:

* `libvirt` and `qemu`
* `virt-manager` *(optional, for GUI management)*
* `wget`, `unxz`

> 💡 You must have root or `sudo` privileges to manage VMs with libvirt.

---

## 🧭 Steps

### 1. Change to the libvirt image directory

```bash
cd /var/lib/libvirt/images/
```

All virtual machine disk images are typically stored here.
We’ll download and unpack the Home Assistant disk image into this location.

---

### 2. Download the Home Assistant OS QCOW2 image

```bash
sudo wget https://github.com/home-assistant/operating-system/releases/download/11.2/haos_ova-16.2.qcow2.xz
```

* Downloads the **Home Assistant OS** release (version `16.2`) from the official GitHub repository.
* The `.xz` file is a compressed version of the QCOW2 disk image.

---

### 3. Extract the image

```bash
sudo unxz haos_ova-16.2.qcow2.xz
```

* Decompresses the downloaded file.
* You’ll get the usable disk image: `haos_ova-16.2.qcow2`.

---

### 4. Stop and remove any existing Home Assistant VM

```bash
sudo virsh destroy homeassistant
sudo virsh undefine homeassistant --nvram
```

* **`destroy`**: Forcefully stops the running VM (if it exists).
* **`undefine`**: Removes the VM definition from libvirt, including its UEFI NVRAM.

> ⚠️ This will not delete the disk file, but it will remove the VM configuration.

---

### 5. Recreate the Home Assistant VM

```bash
sudo virt-install \
  --name homeassistant \
  --memory 4096 \
  --vcpus 2 \
  --disk /var/lib/libvirt/images/haos_ova-16.2.qcow2,format=qcow2,bus=sata \
  --import \
  --os-variant generic \
  --network type=direct,source=eno1,source_mode=bridge,model=virtio \
  --graphics vnc \
  --boot uefi \
  --noautoconsole
```

#### 🔍 Explanation of options:

| Option                                                              | Description                                                                                                      |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `--name homeassistant`                                              | Names the VM “homeassistant”.                                                                                    |
| `--memory 4096`                                                     | Allocates 4 GB of RAM.                                                                                           |
| `--vcpus 2`                                                         | Assigns 2 virtual CPUs.                                                                                          |
| `--disk`                                                            | Specifies the disk image location and settings.                                                                  |
| `--import`                                                          | Uses an existing disk image instead of installing from ISO.                                                      |
| `--os-variant generic`                                              | Generic OS type for compatibility.                                                                               |
| `--network type=direct,source=eno1,source_mode=bridge,model=virtio` | Connects the VM directly to the host’s physical NIC (`eno1`) using a bridged connection for full network access. |
| `--graphics vnc`                                                    | Enables VNC console access.                                                                                      |
| `--boot uefi`                                                       | Boots the VM using UEFI firmware.                                                                                |
| `--noautoconsole`                                                   | Prevents automatic console attachment.                                                                           |

---

## ✅ After Installation

Once the VM is running:

* Access Home Assistant via its web interface (usually at `http://<your_host_ip>:8123`).
* Manage the VM using:

  ```bash
  sudo virsh list --all
  sudo virsh start homeassistant
  sudo virsh shutdown homeassistant
  ```

---

## 🧹 Optional: Verify Network Connectivity

Ensure your host’s interface `eno1` supports direct bridging.
If you encounter network issues, you may need to configure a proper **bridge interface** (e.g. `br0`) in `/etc/netplan/` or NetworkManager.

---

**Author:** *Your Name*
**Based on:** [Home Assistant OS Releases](https://github.com/home-assistant/operating-system/releases)

---

Would you like me to include a **"troubleshooting"** section (e.g. VNC connection tips, bridge errors, etc.) at the bottom?
