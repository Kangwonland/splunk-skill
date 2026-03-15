# limits.conf [search] Stanza Reference

Key parameters for the `[search]` stanza in `$SPLUNK_HOME/etc/system/local/limits.conf`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `base_max_searches` | `6` | Base number of concurrent searches per search head |
| `max_searches_per_cpu` | `1` | Max concurrent searches per CPU core |
| `max_rt_search_multiplier` | `2` | Multiplier for real-time search slots |
| `max_count` | `500000` | Max events kept in RAM per search |
| `max_macro_depth` | `100` | Max nested macro expansion depth |
| `max_subsearch_depth` | `8` | Max nesting depth for subsearches |
| `min_prefix_len` | `1` | Min chars before wildcard for index optimization |
| `fieldstats_update_freq` | `0.0` | Frequency of fieldstats updates (0=end only) |
| `max_searches_perc` | `50` | Max % of search slots for ad-hoc searches |
| `search_process_mode` | `auto` | Search process mode: `auto`, `historical`, `realtime` |
| `write_multifile_results_out` | `0` | Max results written to multi-file output |
| `chunk_size` | `10000` | Events processed per chunk |
| `fetch_multiplier` | `1` | Fetch multiplier for preview results |
| `results_queue_read_timeout_sec` | `10` | Seconds to wait for result chunks |
| `timeout` | `86400` | Default search timeout in seconds (24h) |
| `preview_interval` | `0` | Seconds between preview updates (0=auto) |
| `dispatch_quota_retry` | `4` | Retry attempts when dispatch quota exceeded |
| `dispatch_quota_sleep_ms` | `100` | Milliseconds to sleep between retries |
| `status_cache_size` | `10000` | Max cached search job status entries |

Source: https://help.splunk.com/en/splunk-enterprise/administer/limits-conf/9.4/limits-conf-reference
