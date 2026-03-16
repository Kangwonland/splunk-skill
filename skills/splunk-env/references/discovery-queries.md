# SPL Discovery Queries

Run these queries in Splunk to discover your environment metadata. Copy the
results and paste them back — any format (CSV, tab-separated, one-per-line)
is accepted.

---

## Tier 1 — Required

### Indexes (active data only)

```spl
| eventcount summarize=false index=* | where count > 0 | dedup index | sort -count | table index count
```

Returns: All indexes with at least one event, sorted by event count.

### Sourcetypes

```spl
| metadata type=sourcetypes index=* | table sourcetype totalCount firstTime recentTime
```

Returns: All sourcetypes with event counts and time range.

### Topology

```spl
| rest /services/server/info | table server_roles version numberOfVirtualCores physicalMemoryMB
```

Returns: Server roles (indexer, search head, etc.), version, resource info.

### Index-Sourcetype Mapping

```spl
| tstats count where index=* by index sourcetype | sort -count | table index sourcetype count
```

Returns: Which sourcetypes exist in which indexes (essential for accurate mapping).

---

## Tier 2 — Recommended

### Lookups

```spl
| rest /services/data/transforms/lookups | table title filename type fields_list
```

Returns: All lookup definitions with type (CSV, KV Store, external).

### Macros

```spl
| rest /services/admin/macros | table title definition args
```

Returns: All search macros with their definitions and argument counts.

### Data Models

```spl
| rest /services/datamodel/model | table title acceleration acceleration.earliest_time eai:digest
```

Returns: Data models with acceleration status and retention.

### Hosts (metadata)

```spl
| metadata type=hosts index=* | table host totalCount firstTime recentTime | sort -totalCount
```

Returns: All hosts reporting data, sorted by event count.

### Host-Index-Sourcetype Mapping (top hosts)

```spl
| tstats count where index=* by host index sourcetype | sort -count | head 100 | table host index sourcetype count
```

Returns: Top 100 host-index-sourcetype combinations.

---

## Tier 3 — Optional

### Eventtypes

```spl
| rest /services/saved/eventtypes | table title search tags
```

Returns: Defined eventtypes with their search definitions and tags.

### Installed TAs and Apps

```spl
| rest /services/apps/local | search title=*TA* OR title=*CIM* OR title=Splunk_SA* | table title version disabled
```

Returns: Technology Add-ons, CIM, and Security Essentials apps.

### All Apps (broader view)

```spl
| rest /services/apps/local | search disabled=0 | table title version
```

Returns: All enabled apps.

### KV Store Collections

```spl
| rest /services/kvstore/collectionconfig | table title app
```

Returns: KV Store collection names and owning apps.

### Saved Searches (informational)

```spl
| rest /services/saved/searches | search disabled=0 is_scheduled=1 | table title cron_schedule search | head 20
```

Returns: Top 20 active scheduled searches (useful for understanding workload).

---

## Tips for Users

- **No admin access?** Tier 1 queries work with standard search permissions.
- **Tier 2-3** REST queries may require admin or `list_settings` capability.
- **Large environments**: Add `| head 50` to limit output if results are overwhelming.
- **Paste format**: Any format works — just paste the Splunk output as-is.
