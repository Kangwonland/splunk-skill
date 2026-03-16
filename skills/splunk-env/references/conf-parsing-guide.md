# Conf File Parsing Guide

Safe extraction commands for each Splunk configuration file. Use these when
the user provides access to their `$SPLUNK_HOME` directory.

## Scan Order

1. `$SPLUNK_HOME/etc/system/local/` — system-level settings
2. `$SPLUNK_HOME/etc/apps/*/local/` — per-app customizations (higher priority)
3. `$SPLUNK_HOME/etc/apps/*/default/` — app defaults (lower priority)

When the same stanza appears in multiple locations, `local/` overrides `default/`.

---

## indexes.conf

**Extract**: Index names (stanza names)

```bash
grep '^\[' indexes.conf | sed 's/[][]//g' | grep -v '^default$' | grep -v '^volume:' | sort -u
```

**Exclude**: `[default]`, `[volume:*]` stanzas — these are internal.

**Additional data** (if available in the stanza):
- `homePath` → derive index location (but do NOT store the path itself)
- `frozenTimePeriodInSecs` → derive retention period

---

## props.conf

**Extract**: Sourcetype names (stanza names)

```bash
grep '^\[' props.conf | sed 's/[][]//g' | grep -v '^source::' | grep -v '^host::' | grep -v '^default$' | sort -u
```

**Exclude**: `[source::*]`, `[host::*]`, `[default]` stanzas — these are
source/host-specific overrides, not sourcetype definitions.

**Additional data**:
- `TRANSFORMS-*` lines → link to transforms.conf lookups
- `TIME_FORMAT` / `TIME_PREFIX` → note time parsing configuration

---

## transforms.conf

**Extract**: Lookup definitions (stanzas with `filename =`)

```bash
grep -B1 'filename\s*=' transforms.conf | grep '^\[' | sed 's/[][]//g' | sort -u
```

**Extract lookup filenames**:
```bash
grep 'filename\s*=' transforms.conf | sed 's/.*=\s*//' | sort -u
```

### Security: CRITICAL

**NEVER read or extract these fields from transforms.conf:**
- `external_cmd`
- `external_type`
- Any field containing `password`, `secret`, `token`, or `key`

Only extract: stanza names and `filename` values.

---

## macros.conf

**Extract**: Macro names and definitions

```bash
# Stanza names (macro names)
grep '^\[' macros.conf | sed 's/[][]//g' | sort -u

# Definitions
grep -A1 '^\[' macros.conf | grep 'definition\s*='
```

**Extract argument count**: Check for `args =` field, or count `$arg$`
placeholders in the definition.

---

## eventtypes.conf

**Extract**: Eventtype names and search definitions

```bash
# Stanza names
grep '^\[' eventtypes.conf | sed 's/[][]//g' | sort -u

# Search definitions
grep 'search\s*=' eventtypes.conf
```

---

## tags.conf

**Extract**: Tag assignments

```bash
grep '^\[' tags.conf | sed 's/[][]//g' | sort -u
```

Stanza format is `[eventtype=name]` — extract the eventtype name from the
stanza header.

---

## datamodels.conf

**Extract**: Data model names and acceleration status

```bash
# Stanza names
grep '^\[' datamodels.conf | sed 's/[][]//g' | sort -u

# Acceleration status
grep 'acceleration\s*=' datamodels.conf
```

---

## inputs.conf

**Extract**: Host information from monitor stanzas

```bash
# Monitor stanzas with host overrides
grep -A5 '^\[monitor://' inputs.conf | grep -E '^\[monitor://|^host\s*='
```

Use this to identify key hosts and their associated log sources.

---

## Large Environment Handling

When 30+ indexes are discovered:

1. **Group by prefix**: Extract common prefixes (e.g., `sec_*`, `web_*`, `app_*`)
2. **Present groups** to the user: *"Found 45 indexes in 6 groups: sec (12), web (8), app (15), infra (5), test (3), misc (2). Which groups are relevant?"*
3. **Store only selected groups** in `splunk-env.md`

---

## General Security Rules

1. **NEVER** store file paths (`$SPLUNK_HOME`, `/opt/splunk/...`) in splunk-env.md
2. **NEVER** read or store credentials, tokens, passwords, or keys
3. **NEVER** execute the conf file contents — only parse text
4. Extract **only** metadata: names, definitions, types
5. Always **confirm** extracted data with the user before saving
