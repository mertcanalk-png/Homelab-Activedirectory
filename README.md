# Windows Server 2022 Active Directory Home Lab

A self-directed home lab where I build and operate a small Windows enterprise
environment from scratch to develop hands-on Active Directory, DNS, and identity
skills for entry-level IT support and system administration roles.

> **Scope note:** This is a self-taught home lab built in my own time for
> learning and interview preparation. It is not paid or on-the-job experience.
> Everything here was built, broken, and fixed by me in VirtualBox.

---

## Why I built this

Job ads for IT support and junior sysadmin roles list Active Directory, Group
Policy, and Windows Server as day-one expectations, but they are hard to
practise without access to a real corporate environment. So I built one. This
lab lets me perform the same identity tasks a help desk does daily — creating
users, resetting passwords, handling lockouts, and managing group-based access —
and document each step the way a real technician would.

The project is split by day so each session is a self-contained, dated piece of
work:

- **Day 1 (this page)** — Build the Domain Controller and complete the core
  identity setup.
- **Day 2** — Join a Windows 11 client, run the live lockout lifecycle, and push
  a Group Policy banner to the client. *(See the Day 2 write-up in `/docs`.)*

---

## Lab environment

| Component | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox 7.2 (host: 16 GB RAM) |
| Domain Controller | DC01 — Windows Server 2022 Standard (Eval), Desktop Experience, 4 GB / 2 vCPU / 60 GB |
| Networking | Static IP `10.0.2.15` /24, DNS pointed to self |
| Domain | New forest `corp.lab.local` (NetBIOS: CORP) |

---

## Day 1 — what I did

### 1. Built the Domain Controller

Installed Windows Server 2022 from scratch, set a static IP with DNS pointed at
itself, renamed the server to DC01, installed the AD DS role, and promoted it to
a Domain Controller in a new forest, `corp.lab.local`. DNS installed
automatically as part of the promotion.

![Deployment configuration — new forest](screenshots/day1-01-forest-deployment-config.png)
![DC01 properties overview](screenshots/day1-02-dc-properties-overview.png)

*Why it matters:* a Domain Controller needs a fixed IP and self-DNS **before**
promotion — Active Directory relies on DNS to locate its own services.

### 2. Organised the directory with an OU structure

Built an Organizational Unit tree — `CORP > Users, Groups, Computers,
Departments > IT, Sales` — with accidental-deletion protection left on.

![OU structure](screenshots/day1-03-ou-structure.png)

*Why it matters:* real environments use OUs to organise objects and to target
Group Policy, rather than dropping everything in the default Users container.

### 3. Created a user (the "new starter" ticket)

Created `alex.taylor` in the IT OU using the `firstname.lastname` convention,
with "user must change password at next logon" enabled.

![User created in IT OU](screenshots/day1-04-user-alex-taylor-created.png)

*Why it matters:* this mirrors the standard new-starter account ticket — the
user sets their own password on first login, so the technician never knows it.

### 4. Set an account lockout policy

Set the account lockout threshold to 3 invalid attempts via the Default Domain
Policy.

![Lockout policy set to 3 attempts](screenshots/day1-05-lockout-policy-3-attempts.png)

*Why it matters:* this makes lockouts behave realistically and is the basis for
handling "I'm locked out" tickets (tested live in Day 2).

### 5. Granted access with a security group

Created a Global Security group, `IT-Staff`, and added `alex.taylor` as a member.

![Adding the user to IT-Staff](screenshots/day1-06-add-alex-to-it-staff.png)
![IT-Staff membership confirmed](screenshots/day1-07-it-staff-membership.png)

*Why it matters:* access is granted through group membership, not per-user — the
standard approach in any managed environment.

---

## Skills demonstrated

Active Directory Domain Services · DNS fundamentals · Windows Server 2022 setup ·
static IP configuration · OU design · user lifecycle (create / reset / unlock) ·
account lockout policy · security groups · VirtualBox virtualisation.

## Full write-up

The complete step-by-step Day 1 summary, including the reasoning behind each
task, is in [docs/Home_Lab_Day1_Summary.pdf](docs/).

## Next session

**Day 2** — Build a Windows 11 Pro client, join it to `corp.lab.local`, run the
full lockout lifecycle end to end (forced password change -> real lockout ->
unlock from the DC), and create a Group Policy logon banner applied to the
client. After that: a Microsoft 365 Developer tenant and Entra ID, repeating the
same identity tasks in the cloud.

---

*Built and documented by Mertcan (Matt) Alkaya — Brisbane, QLD.*
