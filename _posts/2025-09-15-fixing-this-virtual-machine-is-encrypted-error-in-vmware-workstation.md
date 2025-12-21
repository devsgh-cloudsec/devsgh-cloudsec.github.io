---
layout: post
title: "Fixing Encrypted VM in VMware Workstation 17.x"
date: 2025-09-15
categories: [VMware, Troubleshooting]
tags: [vmware, encryption, vTPM, vmdk]
---

**VMware Workstation 17.x** introduced a behavior where VMs with an attached **vTPM** may be marked as "encrypted" even if the user did not explicitly encrypt them. If you're seeing a password prompt on VM startup, this guide will walk you through the steps to fix it.

---

## Step-by-Step Fix (Workstation 17.6.1)

### Step 1: Backup

- Make a full copy of your VM folder. Include all `.vmx`, `.vmdk`, `.nvram`, `.vmxf`, `.vmsd`, `.vmsn`, etc.
- If possible, create a checkpoint or ensure you have a working backup of all relevant VMDK files (especially the "flat" or monolithic file).

### Step 2: Identify Disk Type & VM State

- Verify whether your VMDK is a **single file** (monolithic sparse or monolithic flat) or split into multiple files/extents/snapshots.
- Ensure the VM is powered off, not suspended, and VMware Workstation is closed when making file edits.

### Step 3: Edit .vmx File to Remove vTPM / Encryption Metadata

- Navigate to the VM's folder. Make a copy of the `.vmx` file (e.g., `vmname.vmx.bak`).
- Open the `.vmx` file in a text editor.
- Remove or comment out (with a `#` or rename) any lines containing:
  - `vtpm.present = "TRUE"`
  - `managedvm.autoAddVTPM = "software"`
  - `vtpm.ekCSR = "…"`
  - `vtpm.ekCRT = "…"`
  - `encryption.keySafe = "…"`
  - `encryption.encryptedKey = "…"`
  - `encryption.data = "…"`
  - `vmx.encryptionType = "partial"`
  - `nvram = "…"`
  - `managedVM.ID = "…"`
- Save the `.vmx` file.

### Step 4: Delete Auxiliary / Metadata Files

In the VM folder, delete or move aside the following files (if present):

- `*.nvram`
- `*.vmxf`
- `*.vmsd`

Also, remove any snapshot files (`*.vmsn`) if you're okay losing snapshot history. Snapshots can complicate metadata.

### Step 5: If VMDK is Single File & Descriptor Needs Repair (Using dsfo / dsfi)

Only perform this step if Steps 3 & 4 did not fix the prompt and your VMDK is a **single file** (monolithic).

**Create a temporary VM:**

- Create a temporary VM in a separate folder. Name it so you can distinguish it.
- Make its virtual disk the same size as your original VM's VMDK.
- Use the monolithic single-file format (i.e., "Store virtual disk as a single file / monolithic sparse" or "monolithic flat" depending on your original VMDK format).

**Extract clean descriptor:**

- Download the **dsfok** tools (`dsfo.exe` and `dsfi.exe`).
- In the temp VM folder, extract a "clean" descriptor header:
```bash
dsfo.exe "TempVM.vmdk" 0 1536 Metadata-OK.bin
```

This extracts the first 1536 bytes from the clean (unencrypted) monolithic VMDK.

**Inject clean header:**

- Copy `Metadata-OK.bin` into your original VM's folder.
- Use **dsfi.exe** to inject the clean header into your original VMDK:
```bash
dsfi.exe "OriginalVM.vmdk" 0 1536 Metadata-OK.bin
```

### Step 6: Re-import / Start VM

- In VMware Workstation 17.6.1, either remove the VM from the library (if it shows), then open the edited `.vmx` file using **File → Open a Virtual Machine…**.
- Try starting the VM.

### Step 7: Optional: Re-add vTPM if Needed

If you originally needed TPM (for Windows 11, etc.), and want it back after restoring:

- Go to **VM settings → Hardware tab → Add → Trusted Platform Module**.
- If the `.vmx` file needs it, add back a line like:
```bash
managedvm.autoAddVTPM = "software"
```

**Note:** Only do this after the VM is booted, confirmed working, and you understand the consequences of re-adding vTPM.

---

## Special Tips / Things to Watch

- If the `.vmx` file still contains `vmx.encryptionType = "partial"`, you **must** remove that line. This entry is new in VMware Workstation 17.x when vTPM is present.

- If the `.vmx` file or descriptor is **truly encrypted** (not just flagged as encrypted metadata), recovery may be impossible unless you have the password/key. VMware's KB says recreating the descriptor cannot be used for encrypted disk descriptor files.

- Always perform these operations with VMware Workstation **closed**.

- If deleting the `.nvram` file causes issues (like boot firmware or UEFI boot problems), you may need to reconfigure the firmware settings in the `.vmx` file. Add `firmware = "efi"` if appropriate, etc.

---

## References & Inspirations

- [Reddit: "This virtual machine is encrypted..." error](https://www.reddit.com/r/vmware/comments/17brrwg/this_virtual_machine_is_encrypted_you_must_enter/)
- [Reddit: Move a Windows 11 VM with TPM](https://www.reddit.com/r/vmware/comments/tkeu8q/move_a_windows_11_vm_with_tpm_to_a_new_computer/i1szpho/)
- [Reddit Comment: Metadata injection method](https://www.reddit.com/r/vmware/comments/tkeu8q/comment/i1szpho/)
- [VMware Forum: vTPM & Encrypted Descriptors](https://community.broadcom.com/vmware-cloud-foundation/communities/community-home/digestviewer/viewthread?GroupId=7171&MessageKey=fdfa255c-08c8-48e1-b1e3-ba0826e59e2a&CommunityKey=fb707ac3-9412-4fad-b7af-018f5da56d9f)
- [Archived VMware Community Thread on vTPM](https://web.archive.org/web/20220323211631/https://communities.vmware.com/t5/VMware-Workstation-Player/VMware-Player-Try-to-add-virtual-TPM-to-vmx-file-but-VMX-file-is/m-p/2874274#M37862)

---

**Got questions?** Open an issue on [GitHub](https://github.com/devsgh-cloudsec/devsgh-cloudsec.github.io) or leave a comment below.
