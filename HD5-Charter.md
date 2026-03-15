\# HaleDistrict HD5 Charter



\## Purpose



HD5 represents the next major evolution of the HaleDistrict homelab environment. The goal of HD5 is to transition HaleDistrict toward a more professional and reproducible infrastructure model while keeping the environment manageable and buildable by a single administrator.



HD5 emphasizes:



\- Infrastructure reproducibility

\- Script-driven configuration

\- Clean golden images

\- Multi-school network architecture

\- Realistic enterprise features (DFS, folder redirection, pilot rings)

\- Clear documentation and repeatable build practices



\---



\## Design Philosophy



HD5 is intentionally designed as a \*\*controlled step forward\*\* from HD4. Rather than expanding every dimension at once, HD5 focuses on a few high-value improvements:



1\. Script-driven infrastructure

2\. Multi-school network architecture

3\. Centralized script management

4\. Clean golden image deployments

5\. Realistic enterprise services



> \*\*Guiding rule:\*\* Prefer clarity and reliability over complexity.



\---



\## Environment Naming Conventions



HD5 uses consistent naming conventions for machines and infrastructure components.



Machine naming pattern:



HD5-\[ROLE]\[NUMBER]



Examples:



HD5-DC01   Domain Controller  

HD5-FS01   File Server  

HD5-RT01   Router  

HD5-ADM01  Administrative Workstation  



This naming scheme allows environments to be easily distinguished across HaleDistrict builds.



\---



\## Script Architecture



The largest architectural shift in HD5 is the introduction of a structured PowerShell script system.



In HD4, scripts primarily assisted configuration and validation. In HD5, scripts begin to \*\*define and operate the environment itself.\*\*



Scripts will:



\- Configure infrastructure roles

\- Build environment structure

\- Deploy enterprise features

\- Validate environment health

\- Provide remediation tools



This shift moves HaleDistrict toward an \*\*Infrastructure-as-Code mindset\*\*, making the environment reproducible, consistent, faster to rebuild, and easier to expand.



\---



\## Script Execution Model



| Role | Tool / Location |

|---|---|

| \*\*Authoring\*\* | NewThinkPad → VS Code |

| \*\*Storage\*\* | FS01 centralized script repository |

| \*\*Execution\*\* | ADM01 (primary administrative control workstation) |



Typical execution pattern:

```

ADM01

&#x20; ↓

Run PowerShell script

&#x20; ↓

Target machine (DC01 / FS01 / endpoint)

```



\### Script Design Principles



\- Scripts should perform \*\*one clear task\*\*

\- Scripts should be \*\*modular and reusable\*\*

\- Shared logic should live in the \*\*Lib\*\* folder

\- Scripts should avoid hidden side effects

\- Scripts should support \*\*repeatable execution\*\*



\---



\## Script Library Structure



\### Baseline

First-pass configuration of newly deployed machines.

```powershell

HD5-Baseline-Workstation.ps1

HD5-Baseline-ADM01.ps1

HD5-Baseline-ServerCommon.ps1

```



\### Config

Environment-specific configuration scripts.

```powershell

HD5-Config-AD-Core.ps1

HD5-Config-Schools.ps1

HD5-Config-FS01-DDrive.ps1

HD5-Config-FS01-Shares.ps1

HD5-Config-ScriptRepository.ps1

```



\### Features

Enterprise features added to the environment.

```powershell

HD5-Feature-DFSNamespace.ps1

HD5-Feature-PilotRing.ps1

HD5-Feature-LoopbackGPO.ps1

```



\### HealthChecks

Environment validation scripts.

```powershell

HD5-HealthCheck-Core.ps1

HD5-HealthCheck-FS01.ps1

HD5-HealthCheck-RT01.ps1

HD5-HealthCheck-Workstation.ps1

HD5-HealthCheck-DFS.ps1

HD5-HealthCheck-Redirection.ps1

HD5-HealthCheck-All.ps1

```



\### Lib

Shared reusable functions used by other scripts.

```powershell

Get-HD5Config.ps1

HD5-Common-Lib.ps1

HD5-HealthCheck-Lib.ps1

```



\### Remediation

Scripts designed to fix known problems.

```powershell

HD5-Remediate-DNS.ps1

HD5-Remediate-DFS.ps1

HD5-Remediate-WorkstationTrust.ps1

```



\---



\## Core Infrastructure



| Machine | Role |

|---|---|

| \*\*HD5-DC01\*\* | Domain Controller / DNS |

| \*\*HD5-FS01\*\* | File Server / DFS / Script Repository |

| \*\*HD5-RT01\*\* | Ubuntu Router / DHCP / VLAN Routing |

| \*\*HD5-ADM01\*\* | Administrative Workstation |



\### System Role Summary

```

DC01  = Identity / Active Directory

FS01  = Data + DFS + Script Repository

RT01  = Routing + VLAN Segmentation

ADM01 = Administrative Control Workstation

```



\---



\## Network Architecture



HD5 introduces school-based network segmentation.



| VLAN | Purpose |

|---|---|

| VLAN 10 | Core Infrastructure |

| VLAN 20 | High School |

| VLAN 30 | Middle School A |

| VLAN 40 | Middle School B |

| VLAN 50 | Elementary School |



Routing between networks is handled by \*\*RT01 (Ubuntu Server)\*\*.



\---



\## Schools Represented in HD5



\- High School

\- Middle School A

\- Middle School B

\- Elementary School



> Only a small number of endpoints will be deployed initially to keep the environment manageable.



\---



\## Enterprise Services



\### DFS Namespace

FS01 hosts the district DFS namespace.

```

\\\\haledistrict.local\\District

```



\### Folder Redirection

User folders will be redirected to FS01. Data resides on the \*\*D: drive of FS01\*\*.



Folders redirected:

\- Desktop

\- Documents



\### Pilot Ring GPO

HD5 will include at least one pilot ring policy to simulate staged policy deployment.



\### Loopback GPO

HD5 will include at least one loopback processing example, likely used to simulate a school computer lab scenario.



\---



\## FS01 Data Layout



FS01 hosts the primary data storage for the HD5 environment.



Drive layout:



C:  Operating System  

D:  District Data



Example structure:



D:\\District\\

D:\\District\\RedirectedFolders\\

D:\\District\\Scripts\\

D:\\District\\DFSRoots\\



Redirected folders are stored under:



D:\\District\\RedirectedFolders\\Users\\



\---



\## What HD5 Will NOT Attempt



To keep the build achievable, the following features are intentionally postponed to HD6:



\- Full Tier 0 / Tier 1 / Tier 2 administrative model enforcement

\- Large-scale fictional user populations

\- Large GPO policy sets

\- DFS replication

\- Heavy automation of VM deployment



\---



\## HD5 Build Phases



The HD5 environment will be built in the following order.



\### Phase 1 – Core Infrastructure



Deploy core machines:



\- HD5-DC01

\- HD5-FS01

\- HD5-RT01

\- HD5-ADM01



Establish:



\- Domain

\- DNS

\- DHCP

\- base network connectivity



\---



\### Phase 2 – Storage and File Services



Configure FS01:



\- Add D: data drive

\- Create folder structure

\- Configure shared folders

\- Deploy script repository



\---



\### Phase 3 – Script Infrastructure



Establish the script library:



\- Baseline scripts

\- Configuration scripts

\- HealthCheck scripts

\- Remediation scripts



Validate script execution model:



NewThinkPad → FS01 → ADM01 → target machines



\---



\### Phase 4 – Enterprise Services



Deploy enterprise features:



\- DFS Namespace

\- Folder Redirection

\- Pilot Ring GPO

\- Loopback GPO



\---



\### Phase 5 – School Network Segmentation



Configure RT01 VLAN routing and segmentation.



Deploy school networks:



\- High School

\- Middle School A

\- Middle School B

\- Elementary School



\---



\### Phase 6 – Validation



Run the HD5 HealthCheck system to validate:



\- infrastructure health

\- DFS functionality

\- redirected folders

\- network connectivity

\- endpoint configuration



\---



\## Definition of Success



HD5 will be considered successful when:



\- \[ ] Script-driven configuration model established

\- \[ ] Multi-school VLAN architecture functioning

\- \[ ] FS01 hosting redirected folders and DFS namespace

\- \[ ] ADM01 operating as the administrative control station

\- \[ ] Centralized script repository operational

\- \[ ] Pilot ring GPO successfully tested

\- \[ ] Loopback GPO successfully demonstrated

\- \[ ] HealthCheck system expanded and functioning



\---



\## Long-Term Direction



HD5 lays the foundation for future HaleDistrict builds. Possible HD6 expansions include:



\- Additional elementary schools

\- Expanded GPO architecture

\- Tiered administrative security model

\- Larger endpoint populations

\- Further script automation and orchestration

