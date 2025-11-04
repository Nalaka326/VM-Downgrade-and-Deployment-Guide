# VMware VM Downgrade Procedure (ESXi 7 → ESXi 6)

This guide explains how to export a virtual machine from ESXi 7, downgrade it, and deploy it on ESXi 6. Steps include exporting to OVF, removing the manifest (.mf) file, adjusting virtual hardware compatibility, and optionally editing the OVF file to manually change the hardware version.

---

## ✅ Requirements

| Item | Details |
|-----|---------|
| Source ESXi | Version 7.x |
| Destination ESXi | Version 6.x |
| Tools | VMware Workstation / OVF Tool |
| Files Exported | `.ovf`, `.vmdk`, optional `.mf` |

---

## Step 1 — Export the VM from ESXi 7

Export the VM as an **OVF package**:

```
Right-click VM → Export → Export OVF Template
```

This will generate:

```
VM_Name.ovf
VM_Name.vmdk
VM_Name.mf
```

---

## Step 2 — **Delete the `.mf` File**

The `.mf` file contains hash checks.  
Because we will modify the OVF later, this file must be deleted.

```
Delete: VM_Name.mf
```

> If you keep the `.mf` file, deployment will fail with a checksum error.

---

## Step 3 — Import OVF into VMware Workstation

1. Open **VMware Workstation**
2. `File → Open`
3. Select the exported `.ovf`
4. Workstation will prompt to convert → Accept

This step automatically **lowers the virtual hardware version**.

---

## Step 4 — Export from VMware Workstation Back to OVF

```
File → Export to OVF
```

This produces a **new** OVF + VMDK which is compatible with older ESXi versions.

---

## Step 5 — (Optional) Manually Edit the OVF File

If deployment still fails, open the `.ovf` file and locate:

```
<vssd:VirtualSystemType>vmx-xx</vssd:VirtualSystemType>
```

Change the version:

```
vmx-19 → vmx-11  (For ESXi 6 compatibility)
```

Save the file.

---

## Step 6 — Deploy to ESXi 6

Upload converted OVF to ESXi 6 host:

```
Right-click → Deploy OVF Template → Select OVF + VMDK
```

Follow the deployment wizard.

---

## 🎯 Summary

| Step | Action |
|------|--------|
| 1 | Export VM from ESXi 7 |
| 2 | Delete `.mf` file |
| 3 | Import to VMware Workstation |
| 4 | Export from Workstation as OVF |
| 5 | (Optional) Edit OVF to `vmx-11` |
| 6 | Deploy to ESXi 6 |

---

## 📌 Notes

- `.mf` file **must** be removed because its checksums break after conversion.
- VMware Workstation automatically **downgrades hardware compatibility**.
- If still incompatible, manually force hardware version by editing OVF.

---

## ✅ Done!
You can now successfully deploy a VM originally built on ESXi 7 to ESXi 6.
