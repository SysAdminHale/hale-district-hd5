## 2026-03-16

Initial HD5 repository created.

## [2026-03-16] — HD5 Phase 1 — DC01 Deployment (Differencing Disk)

### Objective

Begin HD5 build using image-based deployment. Create first core server (HD5-DC01)
from a verified golden image using differencing disk architecture.

---

### Actions Completed

Verified integrity of `GOLD-WIN-SRV-BUILD.vhdx` via OOBE boot test with no
configuration completed, then restored read-only protection on the golden image.

Created differencing disk `HD5-DC01.vhdx` at `C:\HyperV\VMs\HD5-DC01\`, parented
to `C:\HyperV\GoldenImages\GOLD-WIN-SRV-BUILD.vhdx`. Confirmed initial size of
~4 MB, validating correct linkage to parent.

Created and configured virtual machine `HD5-DC01`:

- Generation: 2
- Memory: 4096 MB
- Disk: Attached existing differencing disk (no new disk created)
- Network: Default Switch (temporary)

Booted VM and completed OOBE as the first real server instance. Renamed system
to `HD5-DC01` via PowerShell:

```powershell
Rename-Computer -NewName "HD5-DC01" -Restart

```

---

### Outcome

Successfully deployed the first HD5 server using differencing disk architecture.
Validated the full end-to-end workflow:

Golden Image → Differencing Disk → VM → OOBE → Rename

Confirmed a clean, fast, and reproducible build pipeline.

---

### Notes / Observations

Differencing disk creation and VM deployment were significantly faster than
traditional install workflows. OOBE behavior confirmed the golden image remains
pristine and reusable. System rename and restart executed quickly and without error.

---

### Next Steps

1. Assign static IP to HD5-DC01
2. Install Active Directory Domain Services (AD DS)
3. Promote HD5-DC01 to Domain Controller
4. Begin defining HD5 domain structure

## 2026-03-17 Server Golden Image investigation

Observed that both the differencing-disk DC VM and a full-clone baseline VM lacked the `ServerManager` PowerShell module and the `Get-WindowsFeature` / `Install-WindowsFeature` cmdlets. This ruled out the differencing-disk workflow as the cause and isolated the issue to the underlying golden image state. GUI-based Server Manager remained available, confirming the OS install was valid but the PowerShell feature-management stack was nonstandard or incomplete. Proceeded with GUI-based role installation to maintain HD5 build momentum. Golden image flagged for rebuild or recapture with validation checks before future use.

## 2026-03-18 HD5 Phase 2 – Domain Controller Deployment & Structure

Configured and promoted primary Domain Controller `HD5-DC01`:

- Assigned static IP configuration
- Installed AD DS role via Server Manager (GUI)
- Promoted server to Domain Controller
- Created new forest: `haledistrict.local`
- DNS installed and integrated automatically
- Rebooted and verified domain login (HALEDISTRICT\Administrator)

Defined initial Active Directory structure via PowerShell:

- Created Organizational Units:
  - HD5-Servers
  - HD5-Workstations
  - HD5-Users
  - HD5-Groups
  - HD5-Admin

---

### Outcome

Successfully deployed and promoted the first HD5 Domain Controller.
Validated full domain bring-up workflow:
Golden Image → Differencing Disk → VM → OOBE → Rename → AD DS Install → Domain Promotion → OU Structure
Confirmed a clean, functional Active Directory environment with DNS integration and structured OU layout.

---

### Notes / Observations

GUI-based AD DS installation used due to missing PowerShell feature-management cmdlets in the golden image. Despite this limitation, Server Manager functioned correctly and allowed successful domain promotion.

OU creation via PowerShell was fast, repeatable, and aligned with desired enterprise-style structure.

Domain immediately usable for identity, authentication, and future service integration.

---

### Next Steps

1. Create and validate initial domain user accounts
2. Redirect default AD containers (Computers / Users)
3. Establish baseline snapshot for DC01
4. Transition to FS01 deployment and domain integration

## 2026-03-18 HD5 Phase 2 – User Creation & Finalization

Created initial domain test user via PowerShell:

```powershell
New-ADUser -Name "Test User" -GivenName "Test" -Surname "User" `
-SamAccountName "tuser" `
-UserPrincipalName "tuser@haledistrict.local" `
-Path "OU=HD5-Users,DC=haledistrict,DC=local" `
-AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
-Enabled $true
```

Updated user credentials and configuration:

Reset password to custom value
Disabled forced password change at next logon

Validated user creation:

Verified via Get-ADUser
Confirmed object placement in ADUC (HD5-Users OU)

Configured domain object placement behavior:

Redirected default containers:
Computers → HD5-Workstations
Users → HD5-Users

powershellredircmp "OU=HD5-Workstations,DC=haledistrict,DC=local"
redirusr "OU=HD5-Users,DC=haledistrict,DC=local"

## Outcome

Successfully validated end-to-end identity workflow: PowerShell → Active Directory → GUI visibility → Object management. Confirmed proper OU targeting and control over default object placement. Established a clean and controlled AD baseline for all future domain-joined systems.

## Notes / Observations

PowerShell-based user creation provides precision and repeatability compared to GUI workflows. Immediate validation via both CLI and ADUC confirms synchronization across management interfaces. Redirection of default containers ensures all new domain objects align with defined OU structure, preventing clutter and maintaining long-term organization. This step represents the transition from "domain exists" to "domain is structured and controlled."

## Next Steps

Create Hyper-V checkpoint: HD5-DC01_BASELINE_CLEAN (completed)
Begin FS01 deployment using differencing disk
Join FS01 to domain and validate connectivity
Design initial file share structure and permissions model
