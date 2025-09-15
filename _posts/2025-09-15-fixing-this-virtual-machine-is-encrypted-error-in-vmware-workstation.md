---
title: "Fix VMware Workstation 17.x 'This virtual machine is encrypted' Error"
date: 2024-01-15 10:00:00 +0000
categories: [Virtualization, VMware]
tags: [vmware, workstation, vtpm, encryption, troubleshooting, fix]
author: your-name
description: "Complete guide to fix the 'This virtual machine is encrypted' error in VMware Workstation 17.x caused by vTPM auto-encryption behavior"
---

VMware Workstation 17.x introduced a behavior where VMs with vTPM attached may be marked as "encrypted" or "partial encryption" even without explicit user encryption. This guide provides a comprehensive step-by-step solution to resolve the password prompt issue.

## Understanding the Issue

<details>
<summary><strong>🔍 What causes this problem?</strong></summary>
<br>
Workstation 17.x introduced a behavior where if a VM has a vTPM attached, VMware may mark it as "encrypted" (or "partial encryption") even if the user didn't explicitly encrypt it.

**Key indicators in .vmx file:**
- `vtpm.present = "TRUE"`  
- `vmx.encryptionType = "partial"`

**Reference:** [Broadcom Knowledge Base - VM Encrypted After Updating](https://knowledge.broadcom.com/external/article/368787/vm-encrpyted-after-updating-to-vmware-wo.html)

**Why this happens:**
- The descriptor (header) of a monolithic single-file VMDK may need replacement if encrypted or corrupt
- Files like `.nvram`, `.vmxf`, `.vmsd` contain metadata that triggers password prompts
- VMware's dsfo/dsfi tools are designed specifically for this issue

**Community Discussion:** [Reddit - VMware Workstation Pro 16.2 vTPM Removal](https://www.reddit.com/r/vmware/comments/qahp5c/vmware_workstation_pro_162_how_to_remove/)
</details>

---

## Step-by-Step Solution

Follow these steps in order. **Test your VM after each major step** - if the password prompt disappears, you can stop there.

<details>
<summary><strong>📋 Step 1: Create Backup</strong></summary>
<br>

**Critical first step - never skip this!**

1. **Make a complete copy** of your VM folder
2. **Include all files:**
   - `.vmx` (configuration)
   - `.vmdk` (disk files)  
   - `.nvram` (BIOS/UEFI settings)
   - `.vmxf` (additional config)
   - `.vmsd` (snapshot data)
   - `.vmsn` (snapshot files)

3. **Verify backup integrity** - ensure all files copied successfully
4. **Test restore process** if you're unsure about backup completeness

> **💡 Pro Tip:** Store backup in a separate location to prevent accidental overwrites
</details>

<details>
<summary><strong>🔎 Step 2: Identify Disk Type & VM State</strong></summary>
<br>

**Before making any changes:**

1. **Verify VMDK type:**
   - **Monolithic:** Single large file (e.g., `VM.vmdk` - one file)
   - **Split:** Multiple files (e.g., `VM.vmdk`, `VM-s001.vmdk`, `VM-s002.vmdk`, etc.)

2. **Ensure proper VM state:**
   - VM must be **powered off** (not suspended)
   - **Close VMware Workstation** completely
   - No VMware processes should be accessing the files

3. **Check for snapshots:**
   - Look for `.vmsn` files in VM folder
   - Snapshots complicate the process - consider removing them if not needed

> **⚠️ Warning:** Never edit VM files while Workstation is running or VM is suspended
</details>

<details>
<summary><strong>✏️ Step 3: Edit .vmx File (Remove vTPM/Encryption Metadata)</strong></summary>
<br>

**Clean the VM configuration file:**

1. **Navigate to VM folder** and create backup:
   ```bash
   copy vmname.vmx vmname.vmx.bak
   ```

2. **Open .vmx in text editor** (Notepad++, VS Code, etc.)

3. **Remove or comment out these lines** (add `#` at beginning or delete entirely):
   ```
   vtpm.present = "TRUE"
   managedvm.autoAddVTPM = "software"
   vtpm.ekCSR = "..."
   vtpm.ekCRT = "..."
   encryption.keySafe = "..."
   encryption.encryptedKey = "..."
   encryption.data = "..."
   vmx.encryptionType = "partial"
   nvram = "..."
   managedVM.ID = "..."
   ```

4. **Save the file** and close editor

> **💡 Note:** The `vmx.encryptionType = "partial"` line is particularly important in 17.x versions
</details>

<details>
<summary><strong>🗑️ Step 4: Delete Auxiliary/Metadata Files</strong></summary>
<br>

**Remove files containing problematic metadata:**

**Delete or move these files from VM folder:**
- `*.nvram` (BIOS/UEFI settings - will be regenerated)
- `*.vmxf` (additional configuration metadata)  
- `*.vmsd` (snapshot metadata)

**Optional - if you don't need snapshots:**
- `*.vmsn` (snapshot files)

**Command examples:**
```bash
# Windows
del *.nvram *.vmxf *.vmsd
del *.vmsn

# Linux/Mac  
rm *.nvram *.vmxf *.vmsd
rm *.vmsn
```

> **⚠️ Important:** Removing `.nvram` will reset BIOS/UEFI settings to defaults
</details>

<details>
<summary><strong>🔧 Step 5: VMDK Descriptor Repair (For Single-File VMDKs Only)</strong></summary>
<br>

**Only perform if Steps 3-4 didn't resolve the issue AND you have a monolithic (single-file) VMDK:**

### 5.1 Create Temporary Clean VM

1. **Create new VM** in VMware Workstation:
   - Same disk size as original
   - Use **"Store virtual disk as a single file"** option
   - Name it clearly (e.g., "TempCleanVM")

### 5.2 Download dsfo/dsfi Tools

1. **Download dsfok tools** containing:
   - `dsfo.exe` (extract descriptor)
   - `dsfi.exe` (inject descriptor)

### 5.3 Extract Clean Descriptor

1. **Navigate to temp VM folder**
2. **Extract clean header:**
   ```cmd
   dsfo.exe "TempVM.vmdk" 0 1536 Metadata-OK.bin
   ```
   - This extracts first 1536 bytes (descriptor header)
   - Creates `Metadata-OK.bin` with clean metadata

### 5.4 Inject Clean Descriptor

1. **Copy `Metadata-OK.bin`** to original VM folder
2. **Inject clean header into original VMDK:**
   ```cmd
   dsfi.exe "OriginalVM.vmdk" 0 1536 Metadata-OK.bin
   ```

> **⚠️ Critical:** This only works for truly single-file VMDKs, not split disks or encrypted descriptors
</details>

<details>
<summary><strong>🚀 Step 6: Re-import & Test VM</strong></summary>
<br>

**Bring your VM back to life:**

1. **Open VMware Workstation**

2. **Remove VM from library** (if still visible):
   - Right-click VM → "Remove from Library"  
   - Don't delete files, just remove reference

3. **Re-import the VM:**
   - File → "Open a Virtual Machine..."
   - Navigate to your edited `.vmx` file
   - Click "Open"

4. **Test startup:**
   - Power on the VM
   - Check if password prompt appears
   - Verify VM boots normally

> **🎉 Success Indicator:** VM starts without password prompt and boots to operating system
</details>

<details>
<summary><strong>🛡️ Step 7: Re-add vTPM (Optional)</strong></summary>
<br>

**If you need TPM functionality (Windows 11, BitLocker, etc.):**

### After VM is working normally:

1. **Access VM Settings:**
   - Right-click VM → "Settings"
   - Go to "Hardware" tab

2. **Add TPM Module:**
   - Click "Add..." → "Trusted Platform Module"
   - Follow wizard to completion

3. **Alternative manual method:**
   - Edit `.vmx` file
   - Add: `managedvm.autoAddVTPM = "software"`
   - **Only do this after confirming VM works without TPM**

> **⚠️ Warning:** Only re-add vTPM after confirming the VM boots successfully without it
</details>

---

## Important Notes & Troubleshooting

<details>
<summary><strong>⚠️ Special Considerations</strong></summary>
<br>

### Critical Points:

1. **Truly Encrypted VMs:**
   - If descriptor is genuinely encrypted (not just flagged), recovery may be impossible without the password/key
   - VMware KB states descriptor recreation cannot work for actually encrypted descriptors

2. **Always Work Offline:**
   - Perform all operations with VMware Workstation completely closed
   - Ensure no VMware processes are running

3. **UEFI/BIOS Settings:**
   - Deleting `.nvram` resets firmware settings
   - May need to re-configure boot settings
   - Add `firmware = "efi"` to `.vmx` if using UEFI

4. **Backup Verification:**
   - Test your backup before starting the process
   - Ensure you can restore if something goes wrong

### If Problems Persist:

- **Double-check** all encryption-related lines were removed from `.vmx`
- **Verify** VM is completely powered off during edits  
- **Confirm** you're working with the correct VMDK type
- **Consider** creating a new VM and importing the fixed VMDK

</details>

---

## References & Resources

- [Broadcom KB: VM Encrypted After Updating to VMware Workstation](https://knowledge.broadcom.com/external/article/368787/vm-encrpyted-after-updating-to-vmware-wo.html)
- [Reddit: VMware Workstation "This virtual machine is encrypted" Error](https://www.reddit.com/r/vmware/comments/17brrwg/this_virtual_machine_is_encrypted_you_must_enter/)
- [Reddit: Move Windows 11 VM with TPM](https://www.reddit.com/r/vmware/comments/tkeu8q/move_a_windows_11_vm_with_tpm_to_a_new_computer/i1szpho/)
- [VMware Community: vTPM & Encrypted Descriptors](https://community.broadcom.com/vmware-cloud-foundation/communities/community-home/digestviewer/viewthread?GroupId=7171&MessageKey=fdfa255c-08c8-48e1-b1e3-ba0826e59e2a&CommunityKey=fb707ac3-9412-4fad-b7af-018f5da56d9f)

---

💬 **Need Help?** Leave a comment below or [open an issue on GitHub](https://github.com/devsgh-cloudsec/devsgh-cloudsec.github.io) if you encounter any problems with this solution.
