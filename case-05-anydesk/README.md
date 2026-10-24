# case-05-anydesk

Defender for Endpoint telemetry from an AnyDesk remote access abuse emulation on two lab
workstations, 2026-06-26 (UTC). Finding the attack in it is the exercise.

## What is inside

| Table | Rows |
|---|---|
| DeviceProcessEvents | 4,153 |
| DeviceNetworkEvents | 8,082 |
| DeviceFileEvents | 37,827 |
| DeviceRegistryEvents | 6,426 |
| DeviceLogonEvents | 61 |
| DeviceEvents | 3,803 |
| DeviceImageLoadEvents | 1,494 |
| AlertEvidence | 24 |

- Hosts: `desktop-ceo.corp.local`, `desktop-hr.corp.local` (domain controller: `john-dc.corp.local`).
- Window: 2026-06-26 00:00 to 2026-06-27 00:00 UTC (AlertEvidence runs one day longer, for late
  alert rows).
- Format: one CSV per Advanced Hunting table, header row, `Timestamp` in UTC
  (`2026-06-26 10:35:11.214961+00:00`).
- `mde/manifest.json`: row counts per table and per host. If your counts differ, the load went
  wrong.
- Parsing notes: some `RegistryValueData` values are `REG_MULTI_SZ` and keep their NUL separators,
  so a tool that strips NULs joins the strings. The domain appears as `corp`, `CORP`, `corp.local`
  and `CORP.local`, exactly as Windows logged it: use case-insensitive operators (`has`, `=~`) on it.

## Load it

Paste `load.kql` into the query editor of an empty database in your free ADX cluster and run it.
It creates all 8 tables from this repo in one go; re-running replaces them, it never duplicates.

## Read this before you hunt

- **The operator IP `172.190.5.225` is real and kept on purpose.** It was my own short-lived test
  VM in Azure, used as the attacker side of the emulation. It is not someone else's infrastructure.
- **My emulation tooling is visible in the data** (for example a PowerShell emulation script).
  That is how the attack was staged in the lab. A real intrusion would not carry it; do not build a
  detection on it.
- `10.0.1.4:7070` is AnyDesk trying a direct connection to the operator VM's Azure private
  address before it falls back to the relay.
- Microsoft and CDN endpoints in the network data are left as they are.

## What was changed (sanitization)

- **Re-mapped one-to-one** (joins still work): domain and domain GUID, local admin name, internal
  IPs, device IDs, SIDs, alert IDs and merge keys. Encoded copies use same-length values. Host names are kept.
- **Removed, 1,323 rows, none from the attack:** Defender's own agent activity (`MsSense.exe` /
  `SenseIR.exe` process trees, `MpCmdRun.exe GetDeviceTicket`) and network rows to the home ISP's
  cache servers. Per-table counts: `mde/manifest.json`.
- Everything else is verbatim; public IPs are real. Browser locale is set to `en_US`.
