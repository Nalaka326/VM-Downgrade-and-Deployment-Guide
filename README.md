# VM Downgrade and Deployment Guide (ESXi 7 → ESXi 6 Using VMware Workstation)

This document explains how to downgrade a virtual machine exported from **ESXi 7** and deploy it on **ESXi 6**.  
The downgrade is required because ESXi 6 does not support the newer VM hardware versions created by ESXi 7.

---

## Overview

| Source Environment | Destination Environment |
|-------------------|------------------------|
| ESXi 7.x          | ESXi 6.x               |
| VM Hardware: vmx-17 / vmx-18 / vmx-19 | VM Hardware Required: vmx-11 to vmx-13 |

Trying to deploy directly may cause errors such as:

The OVF package is not compatible with this version of ESXi.
Access to resource settings on the host is restricted.
Failed to deploy OVF package.

yaml
Copy code

---

## Requirements

| Tool | Purpose |
|------|---------|
| VMware Workstation 12/14/15/16 | Convert VM hardware compatibility |
| Exported VM files (.ova or .ovf + .vmdk) | Source virtual machine |
| Access to ESXi 6 Web UI | Final deployment |

---

## Step 1 — Extract the VM (If It Is a `.ova`)

```bash
tar -xvf exported_vm.ova
This extracts:

Copy code
vm.ovf
vm.vmdk
vm.mf
Step 2 — Delete the .mf File
The .mf manifest file contains SHA checksums that will fail after changes.
Therefore, remove it:

bash
Copy code
rm *.mf
.mf is not required for deployment. Deleting it avoids checksum errors.

Step 3 — Import the VM into VMware Workstation
Open VMware Workstation

Go to File → Open

Select the .ovf file

Import the VM

Step 4 — Downgrade the Hardware Version
Right-click the VM in Workstation

Go to Manage → Change Hardware Compatibility

Select:

nginx
Copy code
Workstation 12.x
(This maps to VM hardware version compatible with ESXi 6)

Confirm and finish

Step 5 — Export Back to OVF
In VMware Workstation:

Go to File → Export to OVF

Save output files:

cpp
Copy code
vm_converted.ovf
vm_converted.vmdk
vm_converted.mf  (optional)
Step 6 — Delete the New .mf File (Recommended)
bash
Copy code
rm vm_converted.mf
Again, removing the manifest avoids deployment checksum validation errors.

Step 7 — (Optional) Fix Hardware Version Manually in .ovf
If deployment still fails:

Open the .ovf file using a text editor.

Find:

php-template
Copy code
<vssd:VirtualSystemType>vmx-19</vssd:VirtualSystemType>
Change it to:

php-template
Copy code
<vssd:VirtualSystemType>vmx-11</vssd:VirtualSystemType>
ESXi Version	Recommended VM Hardware Version
ESXi 6.0	vmx-11
ESXi 6.5	vmx-13
ESXi 6.7	vmx-14

Save the file.

Step 8 — Deploy to ESXi 6
Login to ESXi 6 Web UI

Go to Storage → Datastore Browser

Upload:

vm_converted.ovf

vm_converted.vmdk

Select the .ovf file → Deploy VM from OVF

Complete wizard → Power on VM

Result
Step	Outcome
Deleted .mf	Avoided deployment checksum errors
Downgraded hardware	Ensured compatibility with ESXi 6
Edited .ovf if needed	Resolved version mismatch failures
VM deployed successfully	✅

Notes
Always keep original VM backup

.mf file is optional and usually causes issues when downgrading

Use Workstation 12.x compatibility for best ESXi 6 support

Completed Successfully ✅
