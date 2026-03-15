# SPL2 eval Functions Reference

SPL2 eval functions. Notable differences from SPL1: no Bitwise category
(use `bit_and` etc. as top-level functions); lambda support; explicit types.

## Comparison and Conditional
| Function | Description |
|----------|-------------|
| `case(cond1,val1,...)` | Multi-branch conditional |
| `coalesce(a,b,...)` | First non-null value |
| `if(cond,t,f)` | Ternary conditional |
| `in(value,list...)` | Match any value |
| `ilike(str,pattern)` | Case-insensitive LIKE (`%` wildcard) |
| `like(str,pattern)` | Case-sensitive LIKE (`%` wildcard) |
| `match(str,regex)` | Regex match |
| `null()` | Null value |
| `nullif(a,b)` | Null if a==b |

## Conversion
| Function | Description |
|----------|-------------|
| `tobool(x)` | Convert to boolean |
| `toip(str)` | Convert to IP type |
| `tonumber(str,base)` | Convert to number |
| `tostring(x,fmt)` | Convert to string |
| `totime(str,fmt)` | Parse string to timestamp |
| `typeof(x)` | Return type name |

## Date and Time
| Function | Description |
|----------|-------------|
| `now()` | Current Unix timestamp |
| `relative_time(ts,offset)` | Adjust timestamp |
| `strftime(ts,fmt)` | Format timestamp |
| `strptime(str,fmt)` | Parse time string |

## Informational
| Function | Description |
|----------|-------------|
| `isint(x)` | True if integer |
| `isnull(x)` | True if null |
| `isnotnull(x)` | True if not null |
| `isnum(x)` | True if number |
| `isstr(x)` | True if string |
| `len(str)` | String length |

## JSON
| Function | Description |
|----------|-------------|
| `json_array(...)` | Create JSON array |
| `json_extract(json,key)` | Extract value |
| `json_keys(json)` | Return keys |
| `json_object(k1,v1,...)` | Create JSON object |
| `json_valid(json)` | True if valid JSON |

## Mathematical
| Function | Description |
|----------|-------------|
| `abs(x)` | Absolute value |
| `ceiling(x)` | Round up |
| `floor(x)` | Round down |
| `log(x,base)` | Logarithm |
| `max(a,b,...)` | Maximum |
| `min(a,b,...)` | Minimum |
| `mod(x,y)` | Modulo |
| `pow(x,y)` | Power |
| `round(x,p)` | Round |
| `sqrt(x)` | Square root |

## Multivalue
| Function | Description |
|----------|-------------|
| `map(mv,lambda)` | Apply lambda to each value |
| `mvappend(a,b,...)` | Append to multivalue |
| `mvcount(mv)` | Count values |
| `mvfilter(mv,x)` | Filter with condition |
| `mvindex(mv,i)` | Value at index |
| `mvjoin(mv,delim)` | Join as string |
| `split(str,delim)` | Split to multivalue |

## Text
| Function | Description |
|----------|-------------|
| `lower(str)` | Lowercase |
| `ltrim(str,chars)` | Left trim |
| `replace(str,regex,repl)` | Regex replace |
| `rtrim(str,chars)` | Right trim |
| `substr(str,start,len)` | Substring |
| `trim(str,chars)` | Trim both ends |
| `upper(str)` | Uppercase |
| `urldecode(str)` | URL decode |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4
