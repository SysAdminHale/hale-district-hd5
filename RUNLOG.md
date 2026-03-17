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
