# case-bundles

Sanitized Microsoft Defender for Endpoint telemetry from real attack emulations in the
[Security-in-bits](https://newsletter.securityinbits.com) home lab. Each case is one technique, run on lab hosts, exported from
Defender Advanced Hunting and scrubbed of lab identities. Load a case into a free Azure Data
Explorer (ADX) cluster and hunt it with the same KQL you would run in Defender.

## Cases

| Case | Technique | Hosts | Window (UTC) | Tables |
|---|---|---|---|---|
| [case-05-anydesk](case-05-anydesk/) | AnyDesk remote access abuse (RMM) | 2 | 2026-06-26 | 8 |

## Load a case

You need a Microsoft account. No Azure subscription, no credit card. About 10 minutes.

**Step 1: create a free cluster**

1. Go to [aka.ms/kustofree](https://aka.ms/kustofree) and select **Create cluster**.
2. Give it a display name, set **Database name** to the case name (for example `case05`), and pick
   a location near you.
3. Accept the terms and select **Create**. Provisioning takes a few minutes.

Already have a free cluster? Add a database instead: on the cluster's overview page, under
**Actions**, select **Create database**, select **Create** on the **Create new database** card,
enter the name, then **Create Database**. Use a new, empty database for each case.

**Step 2: load every table in one paste**

1. Open the cluster, select **Query**, and select your database in the left pane.
2. Open the case's `load.kql` in this repo, copy all of it, and paste it into the query editor.
3. Select the whole block and **Run**. It reads each CSV straight from this repo and creates one
   table per file. Each line of the result should say **Completed**.

Running it again is safe: it replaces the tables, it never duplicates rows. **Run it in a new, empty
database:** if a database already has Defender tables with these names, the script replaces them.

**Step 3: check the load**

```kusto
union withsource=T *
| summarize Rows=count(), First=min(Timestamp), Last=max(Timestamp) by T
| order by T asc
```

The row counts must match the case's `mde/manifest.json` (also listed in the case README). A lower
count means a table did not load fully: run `load.kql` again.

Then read the case README: what is inside, what was renamed, what to watch for.

The free cluster is granted for one year. This repo stays your copy of the data, so you can reload
into a new cluster any time.

## How this was made

Every emulation was run by hand in my home lab, and I validated the telemetry myself.
AI-assisted development helped slice the Defender export, scrub the lab
identities and write the load script; I reviewed its output before release. Always verify the
output yourself.
