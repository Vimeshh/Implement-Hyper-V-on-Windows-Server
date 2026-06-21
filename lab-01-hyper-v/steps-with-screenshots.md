# Lab 01 — Implement Hyper-V on Windows Server
### Step-by-Step Walkthrough with Screenshots



---

## Task 1 — Prepare the Environment & Install the Hyper-V Role

### Step 1 — Sign in to SRV01

Sign in to SRV01 as `HEXELO\Administrator`.

![SRV01 Desktop](screenshots/01-srv01-desktop.png)

---

### Step 2 — Prepare for nested Hyper-V (bcdedit)

Open an elevated PowerShell or Command Prompt and run:

```powershell
bcdedit /set hypervisorlaunchtype auto
```

This tells Windows Boot Manager to always launch the hypervisor — required in a nested virtualization environment.

![bcdedit command executed](screenshots/05-bcdedit-command.png)

---

### Step 3 — Restart the server

Immediately after bcdedit, restart the server:

```powershell
Restart-Computer -Force
```

![Restart-Computer command](screenshots/06-restart-computer-powershell.png)

---

### Step 4 — Open Server Manager and launch Add Roles and Features

After the reboot, sign back in and open **Server Manager**. Go to **Manage → Add Roles and Features**.

![Server Manager – Manage menu](screenshots/02-server-manager-manage-menu.png)

---

### Step 5 — Select installation type

On the **Select installation type** page, ensure **Role-based or feature-based installation** is selected. Click **Next**.

![Select installation type](screenshots/03-installation-type.png)

---

### Step 6 — Select destination server

On **Select destination server**, confirm **SRV01.hexelo.com** is selected in the server pool. Click **Next**.

![Select destination server](screenshots/04-select-destination-server.png)

---

### Step 7 — Select the Hyper-V role

On **Select server roles**, check the **Hyper-V** checkbox.

![Select server roles – Hyper-V checked](screenshots/08-select-server-roles-hyperv-checked.png)

---

### Step 8 — Add required features for Hyper-V

A popup appears asking to add management tools required by Hyper-V. Click **Add Features**.

![Add Features popup for Hyper-V](screenshots/07-add-features-hyperv-popup.png)

---

### Step 9 — Review Hyper-V info and advance

The wizard shows the Hyper-V information page. Click **Next** three times to reach the **Create Virtual Switches** page.

![Hyper-V info page](screenshots/09-hyperv-info-page.png)

---

### Step 10 — Select the Ethernet adapter for virtual switch

On the **Create Virtual Switches** page, check the **Ethernet (Microsoft Hyper-V Network Adapter)** checkbox. Click **Next** three times to reach the Confirmation page.

![Create Virtual Switches – Ethernet selected](screenshots/10-create-virtual-switches-ethernet.png)

---

### Step 11 — Enable automatic restart and confirm

Check **Restart the destination server automatically if required**. A dialog appears — click **Yes** to allow automatic restarts.

![Confirm auto-restart dialog](screenshots/11-confirm-auto-restart-dialog.png)

---

### Step 12 — Review and install

Verify the Hyper-V role and all management tools are listed. Click **Install**.

![Confirm installation selections](screenshots/12-confirm-installation-selections.png)

---

### Step 13 — Wait for installation to complete

The installation progress screen shows Hyper-V and all sub-components being installed on SRV01.hexelo.com.

![Installation in progress](screenshots/13-installation-in-progress.png)

---

### Step 14 — Server restarts with Hyper-V

The server reboots automatically. The Hyper-V™ splash screen confirms the hypervisor is now loading during boot.

![Hyper-V boot splash](screenshots/14-hyperv-boot-splash.png)

---

### Step 15 — Confirm installation succeeded

Sign back in after the restart. Open Server Manager — the Add Roles and Features Wizard shows **Installation succeeded on SRV01.hexelo.com**.

![Installation succeeded](screenshots/15-installation-succeeded.png)

> ✅ **Task 1 complete** — Hyper-V role successfully installed on SRV01.

---

## Task 2 — Create a Private Virtual Switch

### Step 16 — Open Hyper-V Manager

Search for **Hyper-V Manager** in the taskbar search and open it.

![Search for Hyper-V Manager](screenshots/16-search-hyperv-manager.png)

---

### Step 17 — Open Virtual Switch Manager and select Private

In Hyper-V Manager, select **SRV01** in the navigation pane. In the **Actions** pane, click **Virtual Switch Manager**.

In the **Virtual Switch Manager for SRV01** window, under **Create virtual switch**, select **Private** and click **Create Virtual Switch**.

> **Private** means VMs can only communicate with each other — no access to the host OS or external network. This is ideal for isolated lab and test environments.

![Virtual Switch Manager – Private selected](screenshots/17-virtual-switch-manager-private-selected.png)

---

### Step 18 — Name the switch and save

In **Virtual Switch Properties**, set the name to `Hexelo Private Switch`. Click **OK**.

The new switch appears in the left pane under Virtual Switches.

![Hexelo Private Switch named](screenshots/18-hexelo-private-switch-named.png)

> ✅ **Task 2 complete** — Private virtual switch `Hexelo Private Switch` created.

---

## Task 3 — Create a Virtual Hard Disk

### Step 19 — Launch the New Virtual Hard Disk Wizard

In Hyper-V Manager, in the **Actions** pane, click **New → Hard Disk**.

![Hyper-V Manager – New Hard Disk option](screenshots/19-hyperv-manager-new-hard-disk.png)

---

### Step 20 — Choose VHDX format

On the **Choose Disk Format** page, select **VHDX**.

> VHDX supports up to 64 TB, has journal-based corruption protection, and is the recommended format for all modern Windows Server workloads. VHD is limited to 2 TB and has no corruption recovery.

![VHD Wizard – VHDX format selected](screenshots/20-vhd-wizard-vhdx-format.png)

---

### Step 21 — Choose Dynamically expanding disk type

On the **Choose Disk Type** page, select **Dynamically expanding**.

> A dynamically expanding disk starts small and only grows on disk as data is written to it — saving storage space. A fixed disk pre-allocates the full size immediately.

![Choose disk type – Dynamically expanding](screenshots/21-choose-disk-type-dynamic.png)

---

### Step 22 — Set name and location

On **Specify Name and Location**, enter:
- **Name:** `OSDisk.vhdx`
- **Location:** `D:\Hyper-V`

![VHD name and location](screenshots/22-vhd-name-location.png)

---

### Step 23 — Configure disk as new blank disk

On the **Configure Disk** page, select **Create a new blank virtual hard disk**. Click **Next**.

![Configure disk – blank](screenshots/23-configure-disk-blank.png)

---

### Step 24 — Review and finish

The summary page confirms the configuration: VHDX, dynamically expanding, OSDisk.vhdx at D:\Hyper-V. Click **Finish**.

![VHD summary – Finish](screenshots/24-vhd-summary-finish.png)

> ✅ **Task 3 complete** — `OSDisk.vhdx` created at `D:\Hyper-V`.

---

## Task 4 — Create a New Virtual Machine (VM3)

### Step 25 — Launch the New Virtual Machine Wizard

In Hyper-V Manager, in the **Actions** pane, click **New → Virtual Machine**.

![Hyper-V Manager – New Virtual Machine](screenshots/25-hyperv-manager-new-vm.png)

---

### Step 26 — Name VM3 and set location

On **Specify Name and Location**:
- **Name:** `VM3`
- Check **Store the virtual machine in a different location**
- **Location:** `D:\Hyper-V`

Click **Next**.

![VM3 name and location](screenshots/26-vm3-name-location.png)

---

### Step 27 — Select Generation 2

On **Specify Generation**, select **Generation 2**.

> Generation 2 VMs use UEFI firmware instead of BIOS, support Secure Boot, can boot from SCSI, and are required for modern Windows Server and Linux workloads. Once set, the generation cannot be changed.

![VM3 – Generation 2 selected](screenshots/27-vm3-generation2.png)

---

### Step 28 — Assign startup memory

On **Assign Memory**, set **Startup memory** to `4096` MB. Click **Next**.

![VM3 – Memory 4096 MB](screenshots/28-vm3-memory-4096mb.png)

---

### Step 29 — Configure networking

On **Configure Networking**, set **Connection** to `Hexelo Private Switch`. Click **Next**.

![VM3 – Networking set to Hexelo Private Switch](screenshots/29-vm3-networking-private-switch.png)

---

### Step 30 — Connect a virtual hard disk

On **Connect Virtual Hard Disk**, select **Create a virtual hard disk**. The wizard auto-populates the name as `VM3.vhdx` in `D:\Hyper-V\VM3\Virtual Hard Disks\`. Click **Next**.

![VM3 – Connect virtual hard disk](screenshots/30-vm3-connect-vhd.png)

---

### Step 31 — Set installation option

On **Installation Options**, select **Install an operating system later**. Click **Next**.

![VM3 – Install OS later](screenshots/31-vm3-install-os-later.png)

---

### Step 32 — Review and finish

The summary confirms: VM3, Generation 2, 4096 MB, Hexelo Private Switch, VM3.vhdx. Click **Finish**.

![VM3 summary – Finish](screenshots/32-vm3-summary-finish.png)

---

### Step 33 — VM3 appears in Hyper-V Manager

VM3 is now listed in the Virtual Machines pane with state **Off**.

![VM3 visible in Hyper-V Manager](screenshots/33-vm3-created-in-manager.png)

---

### Step 34 — Enable Dynamic Memory on VM3

Select VM3, click **Settings** in the Actions pane. Go to **Hardware → Memory**.

Check **Enable Dynamic Memory** and set:
- **Minimum RAM:** `512` MB
- **Maximum RAM:** `6000` MB

Click **OK**.

> Dynamic Memory lets Hyper-V reclaim idle RAM from VM3 and redistribute it to other VMs that need it — maximising host efficiency when running multiple VMs simultaneously.

![VM3 Dynamic Memory settings](screenshots/34-vm3-dynamic-memory-settings.png)

> ✅ **Task 4 complete** — VM3 created (Generation 2, 4 GB startup RAM, Dynamic Memory enabled, connected to Hexelo Private Switch).

---

## Task 5 — Import a Virtual Machine (VM1)

### Step 35 — Launch Import Virtual Machine

In Hyper-V Manager, in the **Actions** pane, click **Import Virtual Machine**. Click **Next** on the Before You Begin page.

![Import VM – Before You Begin](screenshots/35-import-vm-before-you-begin.png)

---

### Step 36 — Locate the VM folder

On **Locate Folder**, enter `D:\Hyper-V\VM1\Virtual Machines`. Click **Next**.

![Import – Locate folder](screenshots/36-import-locate-folder.png)

---

### Step 37 — Select VM1

On **Select Virtual Machine**, VM1 is listed. Select it and click **Next**.

![Import – Select VM1](screenshots/37-import-select-vm1.png)

---

### Step 38 — Choose import type

On **Choose Import Type**, select **Copy the virtual machine (create a new unique ID)**. Click **Next**.

> This creates a fresh identity for the imported VM. If both the original and the copy needed to run on the same host simultaneously, identical IDs would cause conflicts in the hypervisor. Choosing "copy" prevents that.

![Import type – Copy with new unique ID](screenshots/38-import-type-copy-new-id.png)

---

### Step 39 — Choose destination folder

On **Choose Folders for Virtual Machine Files**:
- Check **Store the virtual machine in a different location**
- **Virtual machine configuration folder:** `D:\Hyper-V`

Click **Next**.

![Import – Choose destination](screenshots/39-import-choose-destination.png)

---

### Step 40 — Choose storage folder for VHDs

On **Choose Folders to Store Virtual Hard Disks**, set **Location** to `D:\Hyper-V\VM1\Virtual Hard Disks`. Click **Next**.

![Import – Storage folder for VHDs](screenshots/40-import-storage-folder-vhd.png)

---

### Step 41 — Connect to virtual network

On **Connect Network**, a warning appears: the original switch `vEthernet-HyperV` was not found (expected — it doesn't exist on this host). Set **Connection** to `Hexelo Private Switch`. Click **Next**.

> This is normal when importing a VM from a different host — the original virtual switch name won't match. Reassigning to the correct switch resolves the network configuration error.

![Import – Connect Network to Hexelo Private Switch](screenshots/41-import-connect-network.png)

---

### Step 42 — Review and complete the import

The summary confirms all settings: VM1, Copy (generate new ID), D:\Hyper-V config folder, D:\Hyper-V\VM1\Virtual Hard Disks, Hexelo Private Switch. Click **Finish**.

![Import summary – Finish](screenshots/42-import-summary-finish.png)

---

### Step 43 — Start VM1

VM1 and VM3 are now both visible in Hyper-V Manager. Right-click **VM1** and select **Start**.

![VM1 right-click – Start](screenshots/43-vm1-rightclick-start.png)

---

### Step 44 — Connect to VM1

VM1 is now in **Running** state (4096 MB assigned, uptime counting). Right-click VM1 and select **Connect**.

![VM1 running – Connect](screenshots/44-vm1-running-connect.png)

---

### Step 45 — Verify VM1 is running

The Virtual Machine Connection window opens showing VM1's lock screen — confirming the VM booted successfully.

![VM1 connected – lock screen](screenshots/45-vm1-connected-lockscreen.png)

---

### Step 46 — Turn off VM1

Close the VM Connection window. Right-click **VM1** in Hyper-V Manager and select **Turn Off**.

![VM1 – Turn Off](screenshots/46-vm1-turnoff.png)

> ✅ **Task 5 complete** — VM1 successfully imported, started, connected to, and shut down.

---

## ✅ Lab Complete — Summary

| Task | What Was Done | Result |
|------|--------------|--------|
| 1 | Prepared nested Hyper-V environment with `bcdedit`, installed Hyper-V role via Server Manager | ✅ |
| 2 | Created a Private Virtual Switch named `Hexelo Private Switch` | ✅ |
| 3 | Created a VHDX virtual hard disk `OSDisk.vhdx` at `D:\Hyper-V` | ✅ |
| 4 | Created VM3 (Generation 2, 4 GB RAM, Dynamic Memory 512 MB–6 GB) | ✅ |
| 5 | Imported VM1 with new unique ID, started it, verified it, shut it down | ✅ |

---

## 💡 Key Concepts Reinforced

- **`bcdedit /set hypervisorlaunchtype auto`** must run before Hyper-V works in a nested VM — the hypervisor won't initialise otherwise
- **Private switch** = VM-to-VM only. Internal = VMs + host. External = VMs + host + network
- **VHDX over VHD** always — larger max size, journaled writes, better resilience
- **Generation 2** uses UEFI + Secure Boot — the right default for any modern workload
- **Dynamic Memory** allows Hyper-V to balloon RAM across VMs dynamically — better host utilisation
- **Import with new unique ID** is safe for cloning — prevents VM identity conflicts on the same hypervisor host
