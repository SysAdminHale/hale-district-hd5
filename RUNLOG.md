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

## [2026-03-XX] — GPO Pilot Ring Exploration + Modeling Validation

### Objective

Begin exploring Group Policy design strategies in HD5, with a focus on pilot ring deployment and validation using Group Policy Modeling.

---

### Actions Completed

- Created pilot security group:
  - SG-HD5-Pilot-Workstations (in HD5-Groups OU)

- Created and configured new GPO:
  - GPO-HD5-Pilot-Disable-ControlPanel
  - Purpose: Disable Control Panel access for pilot devices

- Linked GPO to:
  - OU: HD5-Workstations

- Configured Security Filtering:
  - Removed default "Authenticated Users"
  - Added: SG-HD5-Pilot-Workstations

---

### Group Policy Modeling (Simulation)

- Launched Group Policy Modeling Wizard on HD5-DC01
- Simulated:
  - Computer in: HD5-Workstations OU
  - Security Group Membership:
    - SG-HD5-Pilot-Workstations

- Initial Issue:
  - "Name Not Found" error when adding security group
  - Root cause:
    - Incorrect group name used ("HD5-Pilot-Workstations")
  - Resolution:
    - Corrected to: SG-HD5-Pilot-Workstations

---

### Validation Results

- GPO successfully applied in modeling when:
  - Computer is in correct OU AND
  - Computer is member of SG-HD5-Pilot-Workstations

- Confirmed:
  - Security filtering + OU scoping working as expected

---

### Key Learning

- GPO application depends on BOTH:
  - OU location (link scope)
  - Security group membership (filtering)

- Group Policy Modeling is critical for:
  - Pre-deployment validation
  - Avoiding misconfiguration in production

- Naming conventions matter:
  - Consistent "SG-" prefix improves clarity and troubleshooting

---

### Strategic Insight

- This work represents early implementation of:
  - Pilot ring deployment strategy (enterprise best practice)

- HOWEVER:
  - No client workstations currently exist in HD5
  - GPO cannot yet be tested on real endpoints

---

### Decision

- PAUSE further GPO work
- RETURN to core HD5 build sequence:

````plaintext
1. Rebuild GOLD image
2. Sysprep GOLD image
3. Rebuild FS01 from GOLD
4. Deploy client machines (STUD/TEACH)
5. Resume GPO deployment + testing

## [2026-03-XX] — HD5 Closure and Transition to HD6

### Summary
HD5 development revealed key architectural and operational insights but resulted in a partially inconsistent environment due to:

- Golden image contamination via checkpoint (.avhdx)
- Early-stage DC01 build inconsistencies
- Out-of-sequence exploration of GPO and pilot ring strategy prior to client deployment
- Accumulation of test VMs (e.g., BASELINE-INVESTIGATION, TEMP-GOLD-FIX)

---

### Key Lessons Learned

- Golden images must remain:
  - Checkpoint-free
  - Unmodified after finalization
  - Trusted as a clean base

- Hyper-V checkpoints introduce differencing disks (.avhdx):
  - These break golden image integrity if not managed properly

- Proper build sequence is critical:
```plaintext
GOLD → Infrastructure → Clients → GPO
- GPO design (pilot rings) was successfully validated conceptually using Group Policy Modeling, but requires client endpoints for real validation

---

### Decision

HD5 will be:

- Archived to external SSD
- Preserved as a reference and learning milestone

HD5 environment will NOT be continued or repaired

---

### Transition

Begin HD6 with:

- Clean Hyper-V environment
- Clean file structure
- Rebuilt Golden Images from scratch
- Strict adherence to build order and image integrity

---

### Status

HD5:  Closed
HD6:  Initiating

---

### Notes for HD6

- Golden Image must remain checkpoint-free at all times
- Never boot or modify GOLD image outside of controlled preparation workflow
- Use GOLD image ONLY as a source for new VMs or differencing disks
- Maintain strict naming conventions (e.g., SG-, GPO-, HD6- prefixes)
- Follow build sequence without deviation:
  GOLD → Infrastructure → Clients → GPO

  Golden Image Integrity Incident Identified:

- GOLD-WIN-SRV-BUILD.vhdx was found to have an associated .avhdx checkpoint file
- This indicates the image was modified after checkpointing and is no longer a clean base
- Image integrity is considered compromised

Action Taken:
- Image retired and archived to external SSD
- Will not be reused or repaired
- Future builds will enforce checkpoint-free golden image policy
````
