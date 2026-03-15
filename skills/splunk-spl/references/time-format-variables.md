# Time Format Variables (strftime/strptime)

Used in `strftime(timestamp, format)` and `strptime(string, format)` eval functions,
and in `timeformat=` parameters.

## Date Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `%Y` | 4-digit year | `2024` |
| `%y` | 2-digit year | `24` |
| `%m` | Month (01–12) | `03` |
| `%B` | Full month name | `March` |
| `%b` / `%h` | Abbreviated month | `Mar` |
| `%d` | Day (01–31) | `15` |
| `%e` | Day (1–31, space padded) | ` 5` |
| `%j` | Day of year (001–366) | `074` |
| `%u` | Day of week (1=Mon, 7=Sun) | `5` |
| `%w` | Day of week (0=Sun, 6=Sat) | `4` |
| `%A` | Full weekday name | `Friday` |
| `%a` | Abbreviated weekday | `Fri` |
| `%U` | Week number (Sun first, 00–53) | `11` |
| `%W` | Week number (Mon first, 00–53) | `11` |

## Time Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `%H` | Hour 24-hour (00–23) | `14` |
| `%I` | Hour 12-hour (01–12) | `02` |
| `%M` | Minute (00–59) | `30` |
| `%S` | Second (00–59) | `45` |
| `%p` | AM/PM | `PM` |
| `%f` | Microseconds (000000–999999) | `123456` |

## Timezone Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `%Z` | Timezone name | `UTC`, `PST` |
| `%z` | UTC offset | `+0000`, `-0800` |

## Composite
| Variable | Description | Equivalent |
|----------|-------------|------------|
| `%T` | Time HH:MM:SS | `%H:%M:%S` |
| `%D` / `%x` | Date MM/DD/YY | `%m/%d/%y` |
| `%c` | Full datetime | locale-dependent |
| `%+` | Full datetime with timezone | `%a %b %e %T %Z %Y` |
| `%s` | Unix epoch seconds | `1710000000` |
| `%Q` | Unix epoch milliseconds | `1710000000000` |
| `%n` | Newline | |
| `%t` | Tab | |
| `%%` | Literal `%` | |

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-time-range/specify-time-modifiers
