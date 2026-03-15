---
title: Field Expressions and Boolean Logic
impact: CRITICAL
tags: basics, boolean, NOT, field-expression, null
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Field Expressions and Boolean Logic

`NOT field="value"` and `field!="value"` behave differently for null/missing fields.

**Incorrect:**
```spl
index=auth field!="admin"
```
Excludes events where `field` is "admin" but also **excludes events where
`field` does not exist** (null).

**Correct:**
```spl
index=auth NOT field="admin"
```
Returns events where `field` is not "admin" **including** events where
`field` is absent.

**Notes:**
- Use `field::value` (double colon) only for indexed fields — cannot be
  used on search-time extracted fields.
- CIDR notation: `ip="10.0.0.0/24"` matches all IPs in that subnet.
- AND is implicit between terms; OR must be explicit and uppercase.
- See: `references/spl-default-fields.md` for indexed vs search-time fields.
