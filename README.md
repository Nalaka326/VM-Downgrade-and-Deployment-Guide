
# VM Downgrade and Deployment Guide (ESXi 7 → ESXi 6 Using VMware Workstation)

This guide explains how to downgrade a virtual machine exported from **ESXi 7** and deploy it on **ESXi 6**.

---

## Overview

| Source Environment | Destination Environment |
|-------------------|------------------------|
| ESXi 7.x          | ESXi 6.x               |
| VM Hardware: vmx-17 / vmx-18 / vmx-19 | Required VM Hardware: vmx-11 to vmx-13 |

---

## Requirements

| Tool | Purpose |
|------|---------|
| VMware Workstation 12/14/15/16 | Convert VM hardware compatibility |
| Exported VM files (.ova or .ovf + .vmdk) | Source virtual machine |
| ESXi 6 Web UI Access | Deploy final VM |

---

## Step 1 — Extract VM (If File is `.ova`)

```bash
tar -xvf exported_vm.ova
```

This extracts: `.ovf`, `.vmdk`, `.mf`

---

## Step 2 — Delete the `.mf` Manifest File

```bash
rm *.mf
```

This prevents checksum failures when modifying OVF.

---

## Step 3 — Import into VMware Workstation

1. Open VMware Workstation
2. File → **Open**
3. Select `.ovf` and **Import**

---

## Step 4 — Downgrade VM Hardware Version

1. Right-click VM → **Manage**
2. **Change Hardware Compatibility**
3. Select:

```
Workstation 12.x
```

4. Confirm → Finish

---

## Step 5 — Change SCSI Controller (Important)

| Controller Type | Supported in ESXi 6? | Notes |
|-----------------|---------------------|-------|
| VMware Paravirtual | ❌ No | Will fail on import |
| LSI Logic SAS | ✅ Yes | Recommended |
| BusLogic | ⚠ Legacy | Not recommended |

**To change SCSI Controller:**  
Right click VM → Edit Settings → Hardware → **SCSI Controller** → Set to:

```
LSI Logic SAS
```

---

## Step 6 — Export Back to OVF

File → **Export to OVF**

Outputs:

```
vm_converted.ovf
vm_converted.vmdk
vm_converted.mf
```

---

## Step 7 — Delete New `.mf`

```bash
rm vm_converted.mf
```

Avoids import checksum errors.

---

## Step 8 — (Optional) Manually Fix Hardware Version

Edit `.ovf` and replace:

```
<vssd:VirtualSystemType>vmx-19</vssd:VirtualSystemType>
```

with:

```
<vssd:VirtualSystemType>vmx-11</vssd:VirtualSystemType>
```

| ESXi Version | Hardware Version |
|-------------|-----------------|
| ESXi 6.0 | vmx-11 |
| ESXi 6.5 | vmx-13 |
| ESXi 6.7 | vmx-14 |

---

## Step 9 — Deploy to ESXi 6

1. Open ESXi Web UI
2. Storage → Datastore Browser
3. Upload `.ovf` and `.vmdk`
4. Right-click `.ovf` → **Deploy VM**
5. Power on VM

---

## Result

| Step | Status |
|------|--------|
| Manifest removed | ✔ |
| Hardware downgraded | ✔ |
| SCSI Controller set correctly | ✔ |
| VM boots on ESXi 6 | ✅ Success |

---

## Notes

- Keep the original VM as backup
- `.mf` files often cause issues during import
- **LSI Logic SAS** must be used for SCSI to ensure ESXi 6 compatibility
