## 🛠️ Troubleshooting Log

### TrueNAS – Duplicate Hardware Serial Across Proxmox VMs

|  |  |
| --- | --- |
| **Symptom** | TrueNAS instances on different Proxmox nodes reported the same hardware serial, causing identification conflicts. |
| **Root Cause** | VMs cloned/created without explicit disk serials inherit a shared (or missing) identifier. |
| **Fix** | Assign a unique `serial=` to each disk in the VM config. |

**Fix:**

```bash
nano /etc/pve/qemu-server/<vmid>.conf
```

Append a unique `serial=` to each disk line, e.g.:

```ini
scsi0: BUNCE-SSD-01:vm-102-disk-0,iothread=1,size=40G,serial=NAS01
scsi2: BUNCE-HDD-02:vm-102-disk-0,iothread=1,size=900G,serial=NAS03
```

> ⚠️ Serials must be unique per disk _and_ per node — keep them consistent with your hardware inventory so TrueNAS (or any downstream tool) can tell nodes apart at a glance.

No reboot required; the serial is read at boot. Next TrueNAS restart (or disk rescan) picks up the new value.