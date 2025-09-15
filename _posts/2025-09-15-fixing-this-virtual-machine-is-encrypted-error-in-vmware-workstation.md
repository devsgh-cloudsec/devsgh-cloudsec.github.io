Perfect 🙌 I’ll wrap each **Step 1–5** inside `<details>` blocks with a `<summary>` so they’re collapsible in your Chirpy Jekyll blog. That way, readers can expand only the steps they need.

Here’s the updated Markdown:

````md
---
title: 🛠️ "Fixing 'This Virtual Machine is Encrypted' Error in VMware Workstation"
date: 2025-09-15 10:00:00 +1000
categories: [Tutorials, VMware]
tags: [VMware, Encryption, vTPM, Windows 11, Virtual Machine]
description: Step-by-step guide to fix the "This virtual machine is encrypted" error when moving a Windows 11 VM with vTPM in VMware Workstation 17.6+.
---

If you've moved a **Windows 11 virtual machine** (with **vTPM enabled**) to a new PC and suddenly see this:

> ❗ *"This virtual machine is encrypted. You must enter its password to continue."*

...even though **you never encrypted it** — you're not alone. Here's how to fix it. ✅

---

## 📑 Table of Contents

{% include toc %}

---

## 🧠 Why this happens

When you enable **vTPM** in a VM, VMware encrypts certain metadata by default — even if **you didn’t enable full disk encryption**.  
When you move the VM to a different machine, VMware may ask for a **non-existent password**.

---

## 🛠️ What you'll need

- VMware Workstation **Pro 17.6+**
- Your existing **.vmdk** (monolithic/single-file format preferred)
- A **temporary VM** with the same disk size
- Tool: [dsfok.zip](https://sanbarrow.com/files/dsfok.zip) (`dsfo.exe` and `dsfi.exe`)
- A few minutes of patience 😅

---

## 🔧 Step-by-step fix

<details>
<summary>🧹 Step 1: Clean up the VM config</summary>

1. Open the `.vmx` file of your **non-working VM** in a text editor.
2. Remove or comment out these lines if present:

   ```text
   nvram = ...
   managedvm.autoAddVTPM = ...
   managedVM.ID = ...
   encryption.encryptedKey = ...
   vtpm.ekCSR = ...
   vtpm.ekCRT = ...
   vtpm.present = ...
   encryption.keySafe = ...
   encryption.data = ...
````

3. Delete the following files from the VM folder:

   * `*.nvram`
   * `*.vmsd`
   * `*.vmxf`

</details>

---

<details>
<summary>🧪 Step 2: Create a temporary VM</summary>

1. In VMware Workstation, **create a new VM**:

   * Choose **"I will install the OS later."**
   * Use the **same disk size** (e.g., 60 GB).
   * Select **monolithic (single file)** format.

2. Locate the `.vmdk` file of this temporary VM.

</details>

---

<details>
<summary>📦 Step 3: Extract clean VMDK metadata</summary>

1. Download and extract [`dsfok.zip`](https://sanbarrow.com/files/dsfok.zip).
2. Place `dsfo.exe` in the same folder as the **temporary VM**.
3. Run this in Command Prompt:

   ```bash
   dsfo.exe "TEMP_VM.vmdk" 0 1536 Metadata-OK.bin
   ```

   ✅ This extracts unencrypted VMDK metadata.

</details>

---

<details>
<summary>💉 Step 4: Inject clean metadata into the broken VM</summary>

1. Copy `Metadata-OK.bin` to your **non-working VM’s** folder.
2. Copy `dsfi.exe` there too.
3. Run this in Command Prompt:

   ```bash
   dsfi.exe "BROKEN_VM.vmdk" 0 1536 Metadata-OK.bin
   ```

   ✅ This replaces the corrupted/encrypted metadata with a clean one.

</details>

---

<details>
<summary>🔁 Step 5: Reopen the VM</summary>

1. In VMware, select **Open a Virtual Machine** and load the fixed `.vmx`.
2. The **"Encrypted" password prompt** should disappear.
3. If the VM boots — 🎉 success!

</details>

---

## 📝 Optional: Re-enable vTPM

If you want to re-enable vTPM after recovery, add these lines back to `.vmx`:

```text
managedvm.autoAddVTPM = "software"
firmware = "efi"
uefi.secureBoot.enabled = "TRUE"
```

⚠️ Only after confirming the VM runs properly.

---

## 🙌 Final Thoughts

This fix is best for:

* VMs with **vTPM but no BitLocker/full disk encryption**
* Virtual disks stored as **monolithic (single-file)**

If your VM uses split `.vmdk`s or actual encryption, recovery may be trickier.

---

## 📎 References & Inspirations

* [Reddit: "This virtual machine is encrypted..." error](https://www.reddit.com/r/vmware/comments/17brrwg/this_virtual_machine_is_encrypted_you_must_enter/)
* [Reddit: Move a Windows 11 VM with TPM](https://www.reddit.com/r/vmware/comments/tkeu8q/move_a_windows_11_vm_with_tpm_to_a_new_computer/i1szpho/)
* [Reddit Comment: Metadata injection method](https://www.reddit.com/r/vmware/comments/tkeu8q/comment/i1szpho/)
* [VMware Forum: vTPM & Encrypted Descriptors](https://community.broadcom.com/vmware-cloud-foundation/communities/community-home/digestviewer/viewthread?GroupId=7171&MessageKey=fdfa255c-08c8-48e1-b1e3-ba0826e59e2a&CommunityKey=fb707ac3-9412-4fad-b7af-018f5da56d9f)
* [Archived VMware Community Thread on vTPM](https://web.archive.org/web/20220323211631/https://communities.vmware.com/t5/VMware-Workstation-Player/VMware-Player-Try-to-add-virtual-TPM-to-vmx-file-but-VMX-file-is/m-p/2874274#M37862)

---

💬 Got questions? Open an issue on [GitHub](https://github.com/devsgh-cloudsec/devsgh-cloudsec.github.io) or leave a comment below.
