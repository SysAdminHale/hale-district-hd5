\# HaleDistrict HD5 Architecture



This document provides a high-level overview of the HD5 environment.



HD5 simulates a small school district IT infrastructure with centralized services, segmented school networks, and script-driven administration.



\---



\# HD5 Infrastructure Overview



&#x20;                    Internet

&#x20;                       │

&#x20;                       │

&#x20;                  \[ eth0 ]

&#x20;                  HD5-RT01

&#x20;              Ubuntu Router / DHCP

&#x20;                       │

&#x20;                       │

&#x20;                  \[ eth1 ]

&#x20;                       │

&#x20;        ─────────────────────────────────

&#x20;                Core Infrastructure VLAN

&#x20;                     (VLAN 10)

&#x20;        ─────────────────────────────────



&#x20;       ┌───────────────┐

&#x20;       │   HD5-DC01    │

&#x20;       │ Domain Control│

&#x20;       │ DNS / AD DS   │

&#x20;       └───────────────┘



&#x20;       ┌───────────────┐

&#x20;       │   HD5-FS01    │

&#x20;       │ File Server   │

&#x20;       │ DFS Namespace │

&#x20;       │ Script Repo   │

&#x20;       └───────────────┘



&#x20;       ┌───────────────┐

&#x20;       │   HD5-ADM01   │

&#x20;       │ Admin Station │

&#x20;       │ Script Runner │

&#x20;       └───────────────┘





\---



\# School Networks



RT01 routes traffic between school networks and the core infrastructure.



VLAN Segments



VLAN 20  → High School  

VLAN 30  → Middle School A  

VLAN 40  → Middle School B  

VLAN 50  → Elementary School



Example endpoints



High School

&#x20;  STUD-HS01

&#x20;  TEACH-HS01



Middle School A

&#x20;  STUD-MS01

&#x20;  TEACH-MS01



Middle School B

&#x20;  STUD-MS02

&#x20;  TEACH-MS02



Elementary School

&#x20;  STUD-ES01

&#x20;  TEACH-ES01



\---



\# Script Flow Model



Scripts are the primary mechanism for configuring and validating the environment.



Authoring Flow



NewThinkPad

&#x20;  │

&#x20;  ▼

VS Code

&#x20;  │

&#x20;  ▼

Commit scripts to FS01 repository



Execution Flow



ADM01

&#x20;  │

&#x20;  ▼

Run PowerShell script

&#x20;  │

&#x20;  ▼

Target infrastructure



Examples



ADM01 → DC01  (Active Directory configuration)  

ADM01 → FS01  (shares / DFS / storage configuration)  

ADM01 → RT01  (network validation)  

ADM01 → endpoints (baseline / validation)



\---



\# Enterprise Services



Core services deployed in HD5



Active Directory

&#x20;  hosted on DC01



DFS Namespace

&#x20;  \\\\haledistrict.local\\District



Folder Redirection

&#x20;  Hosted on FS01

&#x20;  Data stored on FS01 D: drive



Script Repository

&#x20;  Hosted on FS01

&#x20;  Central script storage for the environment



\---



\# Key Architectural Principles



HD5 follows several core design principles:



• Centralized services  

• Script-driven infrastructure  

• Segmented networks  

• Clean golden image deployments  

• Repeatable environment builds



\---



Internet

&#x20;  │

RT01

&#x20;  │

Core VLAN

&#x20;├── DC01

&#x20;├── FS01

&#x20;└── ADM01

&#x20;  │

School VLANs

&#x20;├── HS

&#x20;├── MS-A

&#x20;├── MS-B

&#x20;└── ES













