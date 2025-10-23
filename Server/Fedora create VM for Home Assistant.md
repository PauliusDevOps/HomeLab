---

# 🏠 Home Assistant Installation via KVM/QEMU (Libvirt)

This guide explains how to install **Home Assistant OS** on a Linux system using **KVM/QEMU (libvirt)**.
We’ll download the official Home Assistant QCOW2 image, remove any existing VM, create a new one, and optionally attach USB devices (e.g., Zigbee/Z-Wave dongles, cameras).

---

## 📦 Prerequisites

Ensure you have the following installed on your system:

* `libvirt` and `qemu`
* `virt-manager` *(optional, for GUI management)*
* `wget`, `unxz`
* `cockpit-machines` *(optional, for USB management via web UI)*

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

| Option                                                              | Description                                                                           |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `--name homeassistant`                                              | Names the VM “homeassistant”.                                                         |
| `--memory 4096`                                                     | Allocates 4 GB of RAM.                                                                |
| `--vcpus 2`                                                         | Assigns 2 virtual CPUs.                                                               |
| `--disk`                                                            | Specifies the disk image location and settings.                                       |
| `--import`                                                          | Uses an existing disk image instead of installing from ISO.                           |
| `--os-variant generic`                                              | Generic OS type for compatibility.                                                    |
| `--network type=direct,source=eno1,source_mode=bridge,model=virtio` | Connects the VM directly to the host’s physical NIC (`eno1`) for full network access. |
| `--graphics vnc`                                                    | Enables VNC console access.                                                           |
| `--boot uefi`                                                       | Boots the VM using UEFI firmware.                                                     |
| `--noautoconsole`                                                   | Prevents automatic console attachment.                                                |

---

## 🧲 Add USB Devices to Home Assistant VM

Home Assistant often requires access to USB devices such as Zigbee/Z-Wave dongles or cameras.
You can attach them using **Cockpit (GUI)** or **command-line**.

---

### ⚙️ Method 1: Using Cockpit (Easiest)

1. **Open Cockpit Machines**

   * Access Cockpit in your browser:
     `https://<your_server_ip>:9090`
   * Go to **Virtual Machines → homeassistant**

2. **Add USB Devices**

   * Click **Add Device → USB Device**
   * Select the devices you want to attach (e.g., your Zigbee stick, camera)
   * Check **Persistent** to make sure they stay attached after reboots.

---

### 🖥️ Method 2: Command Line (Manual)

#### Step 1: List your USB devices

```bash
lsusb
```

Example output:

```
Bus 001 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 003: ID 17ef:482f Lenovo Lenovo 500 RGB Camera
```

Here are the details for our two USB devices:

| Device              | Description                     | Vendor ID | Product ID |
| ------------------- | ------------------------------- | --------- | ---------- |
| Zigbee/Z-Wave stick | Silicon Labs CP210x UART Bridge | `10c4`    | `ea60`     |
| Camera              | Lenovo 500 RGB Camera           | `17ef`    | `482f`     |

---

#### Step 2: Attach USB devices to the running VM

##### Attach Silicon Labs CP210x (Zigbee/Z-Wave stick)

```bash
sudo virsh attach-device homeassistant --file /dev/stdin --persistent <<EOF
<hostdev mode='subsystem' type='usb' managed='yes'>
  <source>
    <vendor id='0x10c4'/>
    <product id='0xea60'/>
  </source>
</hostdev>
EOF
```

##### Attach Lenovo Camera

```bash
sudo virsh attach-device homeassistant --file /dev/stdin --persistent <<EOF
<hostdev mode='subsystem' type='usb' managed='yes'>
  <source>
    <vendor id='0x17ef'/>
    <product id='0x482f'/>
  </source>
</hostdev>
EOF
```

> ✅ The `--persistent` flag ensures the devices stay attached even after a VM reboot.

---

### 🔍 Step 3: Verify in Home Assistant

1. Open Home Assistant → **Settings → System → Hardware**
2. Click **All Hardware**
3. You should see both devices listed.

For the **CP210x** device, it will usually appear as:

```
/dev/ttyUSB0
```

You can use this path when setting up your **Zigbee** or **Z-Wave** integrations.

---

## ✅ After Installation

Once the VM is running:

* Access Home Assistant via your browser:
  **http://<your_host_ip>:8123**
* Manage the VM using:

  ```bash
  sudo virsh list --all
  sudo virsh start homeassistant
  sudo virsh shutdown homeassistant
  ```

---

## 🧹 Optional: Verify Network Connectivity

If the direct network connection fails, you may need to configure a **bridge interface** (e.g., `br0`) in your host network settings using `/etc/netplan/` or NetworkManager.

Would you like me to add a **“Persistent USB Configuration”** section (to ensure USBs auto-reattach after host reboot)?
