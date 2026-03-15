---
title: Form-Template Extraction with kvform
impact: LOW
tags: extraction, kvform, form-template, key-value
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-k/kvform"
---

## Form-Template Extraction with kvform

`kvform` extracts field-value pairs using a template that matches the
structure of form-style log entries (label: value).

**Incorrect:**
```spl
index=form_logs | rex "Name:\s*(?<name>\S+)\s+Email:\s*(?<email>\S+)"
```
Fragile regex breaks when field order changes or spacing differs.

**Correct:**
```spl
index=form_logs | kvform form=contact_form
```
With a `contact_form` stanza in `transforms.conf` defining the template.


**Notes:**
- `kvform` requires a `[contact_form]` stanza defined in `transforms.conf`.
  The stanza specifies the field-value separator and extraction pattern.
- Template format: `field_name: <value_pattern>`
- Less common than `rex`/`spath` — primarily used for structured text forms
  (support tickets, email templates, configuration snippets).
- Consider `rex` with named groups as a simpler alternative for most cases.
