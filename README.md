# Windows Server 2022 Active Directory Home Lab

A self-directed home lab where I build and operate a small Windows enterprise
environment from scratch to develop hands-on Active Directory, DNS, DHCP, and
Group Policy skills for entry-level IT support and system administration roles.

> **Note on scope:** This is a self-taught home lab built in my own time for
> learning. It is not paid or on-the-job experience. Everything here was built,
> broken, and fixed by me in VirtualBox to understand how a real domain
> environment works.

---

## Why I built this

Job ads for IT support and junior sysadmin roles list Active Directory, Group
Policy, and Windows Server as day-one expectations, but these are hard to
practise without access to a real corporate environment. So I built one. This
lab lets me perform the same identity and configuration tasks a help desk or
sysadmin does daily — creating users, resetting passwords, handling lockouts,
and pushing policy to client machines — and document each step the way a real
technician would.

## Lab environment

| Component | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox 7.2 (host: 16 GB RAM) |
| Domain Controller | Windows Server 2022 Standard (Desktop Experience), 4 GB / 2 vCPU / 60 GB |
| Client | Windows 11 Pro |
| Networking | VirtualBox Internal Network, static IPs, isolated (no internet) |
| Domain | `corp.lab.local` (NetBIOS: CORP) |

**Network layout**

| Machine | Role | IP address | DNS |
|---|---|---|---|
| DC01 | Domain Controller | 10.0.2.15 /24 | 127.0.0.1 (self) |
| CLIENT01 | Domain-joined workstation | 10.0.2.20 /24 | 10.0.2.15 |

---

## What I have built so far

### Day 1 & 1b — Domain build and core identity tasks

- Installed Windows Server 2022 from scratch and assigned a static IP.
- Promoted the server to a **Domain Controller**, creating the forest
  `corp.lab.local`.
- Built an **OU structure** (CORP > Users, Groups, Computers, Departments >
  IT, Sales) to mirror how a real business organises its directory.
- Created user `alex.taylor` in the IT OU and a global security group
  `IT-Staff`.
- Configured an **account lockout policy** (3 failed attempts) via the Default
  Domain Policy.
- Joined **CLIENT01** (Windows 11) to the domain.
- **Live-tested the full identity lifecycle:** forced a password change on first
  logon, triggered a real account lockout with 3 bad passwords, then unlocked
  the account from DC01 using Active Directory Users and Computers (ADUC).
- Created and linked a **Group Policy Object** (a logon banner) to the CORP OU,
  forced it with `gpupdate /force`, and confirmed it applied on CLIENT01.

See [docs/Day1-1b.pdf](docs/) for the full step-by-step write-up with screenshots.

---

## Problems I hit and how I solved them

Documenting the failures matters more than the successes — troubleshooting is
the actual skill.

- **VirtualBox black screen / no boot** — traced to UEFI boot order; corrected
  the firmware boot sequence.
- **Windows 11 setup blocked** — the install required TPM 2.0 and Secure Boot;
  enabled both in the VM settings.
- **Forced Microsoft account sign-in during OOBE** — bypassed by cutting the
  VM's internet during setup to allow a local account, a common real-world
  workaround.
- **GPO not applying immediately** — learned that GPOs refresh on a ~90-minute
  cycle and at boot/logon; `gpupdate /force` applies them on demand.

---

## Key things I learned

- Static IPs on an isolated internal network are the reliable baseline for lab
  connectivity.
- A Domain Controller must point its DNS at itself — Active Directory depends on
  DNS to locate services.
- Account lockout and password policies are set at the domain level and apply to
  every user.
- Unlocking a locked account and forcing a password reset are two of the most
  common real help desk tasks — I've now done both end to end.

---

## Planned next steps

- **Day 2:** Extend into the cloud — set up a Microsoft 365 Developer tenant and
  repeat the same identity tasks (user creation, password reset, lockout) in
  **Entra ID**, then explore hybrid identity concepts.
- Broaden into Microsoft 365 administration and ticketing workflows.

---

*Built and documented by Mertcan (Matt) Alkaya — Brisbane, QLD.*
