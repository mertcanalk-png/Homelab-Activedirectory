# Troubleshooting Log — Time, Timezone & DC Networking

> Self-directed home lab. Problems I hit while running my own domain, and how I
> worked through them. Not paid work.

This log captures a tangle of issues that came up around time synchronisation on
the lab — the kind of multi-layered problem where the symptom ("the clocks don't
match") had several possible causes that had to be separated one at a time.

The environment: a Windows Server 2022 domain controller (DC01, running AD DS and
DNS) and a domain-joined Windows 11 client (CLIENT01), both in VirtualBox.

![DC01 running AD DS and DNS, with the client alongside](screenshots/lab-dc01-adds-dns-running.png)

---

## The trigger

While practising a "user's clock is wrong" scenario, I went to verify time
behaviour on my own domain and found the client (CLIENT01) and the domain
controller (DC01) were showing different times. What looked like one problem
turned out to be several stacked together.

---

## Problems, in the order I untangled them

### 1. The clocks didn't match — actual cause: timezone

**Symptom:** DC01 and CLIENT01 showed different times.

**What I learned to check first:** a time *difference* has more than one possible
cause, and they show up differently:

- A **whole-hour** gap usually means the two machines are in **different
  timezones** — they can be perfectly synced and still display different times.
- A **few-minutes** gap usually means **clock drift / broken sync**.

**Root cause:** the two VMs were set to different timezones. Once I set both to the
same timezone, the displayed times matched.

**Lesson:** check the simple, obvious thing (timezone) before diving into time-sync
internals. I went deep on `w32tm` and the domain time hierarchy first — the actual
fix was a timezone setting. Measure the size of the gap before choosing a theory.

### 2. DC01 lost its static IPv4

**Symptom:** DC01's IPv4 static configuration had reverted (it had picked up an
auto-assigned address instead of its fixed one).

**Why it matters:** a domain controller runs DNS, and every domain machine finds
the domain by the DC's IP. If the DC's address changes or goes dynamic, clients
can't locate the domain. A DC must have a static IP — this is foundational, not
optional.

**Fix:** confirmed the current state with `ipconfig /all`, then restored the static
IPv4 settings in the adapter's IPv4 properties (address, subnet, DNS pointed at
itself), matching the values from my Day 1 build.

**Lesson:** always read the live config off the machine with `ipconfig /all` rather
than trusting notes — and a DC losing its static IP breaks name resolution for the
whole domain.

### 3. Access Denied running w32tm

**Symptom:** `w32tm /query /configuration` returned "Access Denied."

**Cause:** the command needs administrator rights and the window wasn't elevated.

**Fix / real-world method:** right-click Command Prompt -> "Run as administrator"
and provide admin credentials at the UAC prompt. In a real environment a technician
uses a **separate admin account** for this — standard users (and a tech's own daily
login) don't hold admin rights. Elevate the single task, do the work, close it.
This is the principle of **least privilege**.

---

## How domain time is *supposed* to work (what I learned)

Time in a domain flows in a chain:

**Reliable external source** -> **PDC Emulator** (one authoritative DC) -> **other
DCs** -> **domain-joined clients**

I confirmed the client end of this chain is working. Running `w32tm /query /status`
on CLIENT01 shows it is syncing from the domain controller, not from the internet
or the host:

![Client syncing time from DC01](screenshots/timesync-client-synced-from-dc.png)

The lines that prove it: `Source: DC-01.corp.lab.local` and
`ReferenceId ... source IP: 10.0.2.15` (DC01's address). `Stratum: 2` means the
client sits one level below the DC in the time hierarchy — exactly as it should.

On the DC, `w32tm /query /configuration` confirms the correct mode for a domain
machine — `Type: NT5DS` (sync from the domain hierarchy, not a manually set NTP
server):

![DC time configuration showing NT5DS](screenshots/timesync-dc-config-nt5ds.png)

Key facts I picked up:
- A domain-joined client should sync from the domain, not directly from the
  internet.
- Clocks more than **5 minutes** off from the DC break **Kerberos
  authentication** — so time is a security dependency, not cosmetic.
- Useful commands: `w32tm /query /status` (where am I getting time, how far off),
  `w32tm /query /configuration` (full config; `Type` should be `NT5DS` for a domain
  machine), `w32tm /resync` (force a resync), `netdom query fsmo` (find the PDC
  Emulator).
- In an **isolated lab with no internet**, the DC has no external source to reach,
  so it shows `Local CMOS Clock` / `Free-running System Clock` as its source, and
  Windows reports "no internet access" — this is expected and normal for a closed
  lab. In production the DC would point at an external time source.

---

## Skills demonstrated

Time-sync troubleshooting · timezone vs drift vs sync isolation · `w32tm` ·
`ipconfig /all` · restoring a DC's static IP · Run as administrator / least
privilege · understanding the domain time hierarchy and its link to Kerberos.

---

*Documented by Mertcan (Matt) Alkaya — Brisbane, QLD.*
