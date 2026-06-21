# Lab 01 — Implement Hyper-V on Windows Server

**Lab Reference:** WSVHA.1-001  
**Platform:** [Learn on Demand — Lab 76720](https://alh.learnondemand.net/Lab/76720?instructionSetLang=en&classId=694960)  
**Duration:** ~30 minutes  
**Status:** ✅ Completed  
**AZ-800 Domain:** Manage virtual machines and containers

---

## 🎯 Objective

Install and configure Hyper-V on a Windows Server machine (SRV01) in a nested virtualization environment. Create and manage virtual networking, virtual hard disks, and virtual machines — both from scratch and via import.

---

## 🌍 Why This Matters (Real-World Context)

Hyper-V is Microsoft's native hypervisor built into Windows Server. In enterprise environments it powers:

- **Server consolidation** — running multiple workloads on fewer physical machines
- **Test/Dev isolation** — spinning up isolated VMs without extra hardware
- **Hybrid scenarios** — on-premises VMs that replicate to Azure Site Recovery
- **Private networking** — internal-only VM communication with no external exposure

Understanding Hyper-V is foundational for any Windows Server or hybrid cloud administrator role.

---

## 🖥️ Environment

| Item | Value |
|------|-------|
| Server | SRV01 |
| Domain | HEXELO.COM |
| Account | HEXELO\Administrator |
| OS | Windows Server (nested Hyper-V) |
| Storage Drive | D:\Hyper-V |

---

## 📌 Tasks Completed

### Task 1 — Prepare for Nested Hyper-V & Add the Hyper-V Role

**What I did:**

Before installing Hyper-V in a nested virtualization environment (a VM running inside another VM), the hypervisor launch type must be set explicitly — otherwise the inner Hyper-V won't initialize correctly.

Ran the following in an elevated PowerShell prompt:

```powershell
bcdedit /set hypervisorlaunchtype auto
Restart-Computer -Force
```

After the reboot, installed the Hyper-V role via **Server Manager**:

1. Opened **Server Manager → Add Roles and Features**
2. Selected **Role-based or feature-based installation**
3. Chose destination server: `SRV01.hexelo.com`
4. Checked the **Hyper-V** role → accepted required features
5. On the **Create Virtual Switches** page, selected the **Ethernet (Microsoft Hyper-V Network Adapter)**
6. Enabled **Restart the destination server automatically if required**
7. Completed the install and waited for the server to restart

**Why `bcdedit /set hypervisorlaunchtype auto`?**  
In a nested environment, the physical host's hypervisor must expose virtualization extensions to the guest OS. The `bcdedit` command tells the Windows Boot Manager to always launch the hypervisor. Without it, Hyper-V installs but VMs fail to start.

**Alternative method (PowerShell):**
```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

---

### Task 2 — Create a Private Virtual Switch

**What I did:**

1. Opened **Hyper-V Manager**
2. Selected **SRV01** in the navigation pane
3. Went to **Actions → Virtual Switch Manager**
4. Selected switch type: **Private**
5. Named it: `Hexelo Private Switch`
6. Clicked **OK**

**Virtual switch types — key differences:**

| Type | Can talk to host? | Can talk to external network? | Use case |
|------|:-----------------:|:-----------------------------:|----------|
| External | ✅ | ✅ | VMs need internet/LAN access |
| Internal | ✅ | ❌ | VMs + host communicate, no external |
| **Private** | ❌ | ❌ | VM-to-VM only, fully isolated |

**Why Private here?**  
The lab simulates an isolated internal environment — VMs communicating only with each other, no external traffic. This is a common pattern for test lab networks and secure workloads.

---

### Task 3 — Create a Virtual Hard Disk (VHD)

**What I did:**

1. In Hyper-V Manager → **Actions → New → Hard Disk**
2. Configured with these settings:

| Property | Value |
|----------|-------|
| Disk Format | **VHDX** |
| Disk Type | **Dynamically expanding** |
| Name | `OSDisk.vhdx` |
| Location | `D:\Hyper-V` |
| Content | New blank virtual hard disk |

**VHDX vs VHD:**

| Feature | VHD | VHDX |
|---------|-----|------|
| Max size | 2 TB | 64 TB |
| Corruption protection | ❌ | ✅ (journal-based) |
| Performance on large blocks | Lower | Higher |
| Recommended for | Legacy | Modern workloads |

**Dynamic vs Fixed disks:**  
A dynamically expanding disk starts small and grows as data is written — saving storage space. A fixed disk pre-allocates the full size immediately, offering slightly better performance but consuming space upfront.

**PowerShell equivalent:**
```powershell
New-VHD -Path "D:\Hyper-V\OSDisk.vhdx" -SizeBytes 50GB -Dynamic
```

---

### Task 4 — Create a Virtual Machine (VM3)

**What I did:**

Used the **New Virtual Machine Wizard** in Hyper-V Manager:

| Property | Value |
|----------|-------|
| Name | `VM3` |
| Location | `D:\Hyper-V` |
| Generation | **Generation 2** |
| Startup Memory | `4096 MB` |
| Network Connection | `Hexelo Private Switch` |
| Virtual Hard Disk | Create a new VHD |
| OS Installation | Install an operating system later |

After creation, enabled **Dynamic Memory** on VM3:

| Setting | Value |
|---------|-------|
| Minimum RAM | `512 MB` |
| Maximum RAM | `6000 MB` |

**Generation 1 vs Generation 2:**

| Feature | Gen 1 | Gen 2 |
|---------|-------|-------|
| Firmware | BIOS | UEFI |
| Secure Boot | ❌ | ✅ |
| Boot from SCSI | ❌ | ✅ |
| PXE Boot (IPv6) | ❌ | ✅ |
| Recommended for | Legacy OS | Windows Server 2012+ / Linux |

**Why Dynamic Memory?**  
Dynamic Memory allows Hyper-V to reclaim unused RAM from idle VMs and give it to VMs that need it. In a host running multiple VMs, this significantly improves overall resource utilization.

**PowerShell equivalent:**
```powershell
New-VM -Name "VM3" -Path "D:\Hyper-V" -Generation 2 -MemoryStartupBytes 4GB -SwitchName "Hexelo Private Switch" -NewVHDPath "D:\Hyper-V\VM3.vhdx" -NewVHDSizeBytes 50GB

Set-VMMemory -VMName "VM3" -DynamicMemoryEnabled $true -MinimumBytes 512MB -MaximumBytes 6GB
```

---

### Task 5 — Import a Virtual Machine (VM1)

**What I did:**

Used **Import Virtual Machine** to bring in a pre-existing VM:

| Property | Value |
|----------|-------|
| Source Folder | `D:\Hyper-V\VM1\Virtual Machines` |
| VM Selected | `VM1` |
| Import Type | **Copy the virtual machine (create a new unique ID)** |
| Config Folder | `D:\Hyper-V` |
| VHD Location | `D:\Hyper-V\VM1\Virtual Hard Disks` |
| Network | `Hexelo Private Switch` |

After import:
- **Started VM1** via Actions → Start
- **Connected** to VM1 to verify it was running
- **Turned off VM1** via Actions → Turn Off

**Import types explained:**

| Type | New ID? | Moves files? | Use case |
|------|:-------:|:------------:|----------|
| Register in place | ❌ | ❌ | Files already in final location |
| Restore | ❌ | ✅ | Replace a deleted VM |
| **Copy (new ID)** | ✅ | ✅ | Clone/duplicate a VM |

Choosing **Copy with new unique ID** ensures the imported VM gets a fresh identity — critical when both the original and clone need to run on the same host simultaneously (avoids UUID conflicts in the hypervisor).

---

## ✅ Summary of What Was Accomplished

| Task | Result |
|------|--------|
| Prepared nested Hyper-V environment | ✅ |
| Installed Hyper-V role via Server Manager | ✅ |
| Created Private Virtual Switch (`Hexelo Private Switch`) | ✅ |
| Created VHDX virtual hard disk (`OSDisk.vhdx`) | ✅ |
| Created VM3 (Gen 2, Dynamic Memory) | ✅ |
| Imported VM1 with new unique ID | ✅ |
| Started, connected to, and shut down VM1 | ✅ |

---

## 💡 Key Takeaways

1. **`bcdedit /set hypervisorlaunchtype auto`** is required before Hyper-V works in a nested VM — without it the hypervisor won't initialize.
2. **Private switches** create a fully isolated network — no host, no internet. Use Internal when the host also needs to participate.
3. **VHDX is always preferred over VHD** for any modern workload — better resilience, larger max size, journaled writes.
4. **Generation 2 VMs** use UEFI and Secure Boot — the right default for anything running Windows Server 2012+ or modern Linux.
5. **Dynamic Memory** doesn't mean less reliable — Hyper-V manages the balloon driver to allocate and reclaim RAM gracefully.
6. **Import with new unique ID** is the safe choice for cloning — it prevents VM identity conflicts on the same host.

---

## 📸 Screenshots

> Screenshots of each task are stored in the [`./screenshots/`](./screenshots/) folder.

---

## 🔗 References

- [Hyper-V on Windows Server — Microsoft Docs](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-on-windows-server)
- [Install-WindowsFeature cmdlet](https://learn.microsoft.com/en-us/powershell/module/servermanager/install-windowsfeature)
- [Virtual Hard Disk types](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/about-dump-encryption)
- [Hyper-V Dynamic Memory](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831766(v=ws.11))
- [Generation 1 vs Generation 2 VMs](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
