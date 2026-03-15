# SPL eval Functions Reference

## Bitwise Operations
| Function | Description |
|----------|-------------|
| `bit_and(x,y)` | Bitwise AND |
| `bit_or(x,y)` | Bitwise OR |
| `bit_not(x)` | Bitwise NOT |
| `bit_xor(x,y)` | Bitwise XOR |
| `bit_shift_left(x,n)` | Left shift |
| `bit_shift_right(x,n)` | Right shift |

## Comparison and Conditional
| Function | Description |
|----------|-------------|
| `case(cond1,val1,cond2,val2,...)` | Multi-branch conditional |
| `coalesce(a,b,...)` | First non-null value |
| `if(cond,true_val,false_val)` | Ternary conditional |
| `in(value,list...)` | Match any value in list |
| `like(str,pattern)` | SQL-like pattern match (`%` wildcard) |
| `match(str,regex)` | Regex match, returns true/false |
| `null()` | Returns null value |
| `nullif(a,b)` | Returns null if a==b, else a |
| `validate(cond,msg,...)` | Returns first failing message |

## Conversion
| Function | Description |
|----------|-------------|
| `ipmask(mask,ip)` | Apply subnet mask to IP |
| `num(x)` | Convert to number |
| `printf(fmt,...)` | Format string |
| `substr(str,start,len)` | Substring |
| `tostring(x,fmt)` | Convert to string (fmt: `"hex"`, `"commas"`, `"duration"`) |
| `tonumber(str,base)` | Convert string to number |
| `typeof(x)` | Returns type name |

## Cryptographic
| Function | Description |
|----------|-------------|
| `md5(str)` | MD5 hash |
| `sha1(str)` | SHA-1 hash |
| `sha256(str)` | SHA-256 hash |
| `sha512(str)` | SHA-512 hash |

## Date and Time
| Function | Description |
|----------|-------------|
| `now()` | Current Unix timestamp |
| `relative_time(ts,offset)` | Adjust timestamp by offset |
| `strftime(ts,fmt)` | Format timestamp as string |
| `strptime(str,fmt)` | Parse string to timestamp |
| `time()` | Current time (same as now()) |

## Informational
| Function | Description |
|----------|-------------|
| `isint(x)` | True if x is an integer |
| `isnotnull(x)` | True if x is not null |
| `isnull(x)` | True if x is null |
| `isnum(x)` | True if x is a number |
| `isstr(x)` | True if x is a string |
| `len(str)` | String length |

## JSON
| Function | Description |
|----------|-------------|
| `json_array(...)` | Create JSON array |
| `json_array_to_mv(json_arr)` | JSON array to multivalue field |
| `json_extend(json,...)` | Extend JSON object |
| `json_extract(json,key)` | Extract value from JSON |
| `json_extract_exact(json,key)` | Extract with exact key match |
| `json_keys(json)` | Return keys of JSON object |
| `json_make_dict(k1,v1,...)` | Create JSON object |
| `json_object(k1,v1,...)` | Create JSON object (alias) |
| `json_set(json,key,val)` | Set value in JSON |
| `json_set_exact(json,key,val)` | Set with exact key |
| `json_valid(json)` | True if valid JSON |

## Mathematical
| Function | Description |
|----------|-------------|
| `abs(x)` | Absolute value |
| `ceiling(x)` | Round up to integer |
| `exp(x)` | e^x |
| `floor(x)` | Round down to integer |
| `ln(x)` | Natural logarithm |
| `log(x,base)` | Logarithm (default base 10) |
| `max(a,b,...)` | Maximum value |
| `min(a,b,...)` | Minimum value |
| `mod(x,y)` | Modulo |
| `pi()` | π constant |
| `pow(x,y)` | x^y |
| `random()` | Random integer (0 to 2^31-1) |
| `round(x,precision)` | Round to precision |
| `sigfig(x)` | Round to significant figures |
| `sqrt(x)` | Square root |

## Multivalue
| Function | Description |
|----------|-------------|
| `commands(str)` | Extract SPL commands from string |
| `mvappend(a,b,...)` | Append values to multivalue field |
| `mvcount(mv)` | Count values in multivalue field |
| `mvdedup(mv)` | Remove duplicates from multivalue field |
| `mvfilter(mv,x)` | Filter multivalue field with eval expr |
| `mvfind(mv,regex)` | Find first match in multivalue field |
| `mvindex(mv,start,end)` | Return value at index |
| `mvjoin(mv,delim)` | Join multivalue field as string |
| `mvmap(mv,expr)` | Apply eval expr to each MV value |
| `mvrange(start,end,step)` | Generate range as multivalue |
| `mvsort(mv)` | Sort multivalue field |
| `mvzip(mv1,mv2,delim)` | Zip two multivalue fields |
| `split(str,delim)` | Split string to multivalue |

## Statistical (eval)
| Function | Description |
|----------|-------------|
| `avg(...)` | Average of values |
| `max(...)` | Maximum of values |
| `min(...)` | Minimum of values |

## Text
| Function | Description |
|----------|-------------|
| `cidrmatch(cidr,ip)` | True if IP in CIDR range |
| `exact(expr)` | Prevent floating-point truncation |
| `lower(str)` | Lowercase string |
| `ltrim(str,chars)` | Remove leading characters |
| `replace(str,regex,replace)` | Regex-based replace |
| `rtrim(str,chars)` | Remove trailing characters |
| `spath(input,path)` | Extract JSON/XML value |
| `split(str,delim)` | Split string to multivalue |
| `substr(str,start,len)` | Substring extraction |
| `trim(str,chars)` | Trim characters from both ends |
| `upper(str)` | Uppercase string |
| `urldecode(str)` | URL-decode a string |

## Trig and Hyperbolic
| Function | Description |
|----------|-------------|
| `acos(x)` | Arccosine |
| `acosh(x)` | Hyperbolic arccosine |
| `asin(x)` | Arcsine |
| `asinh(x)` | Hyperbolic arcsine |
| `atan(x)` | Arctangent |
| `atan2(y,x)` | Two-argument arctangent |
| `atanh(x)` | Hyperbolic arctangent |
| `cos(x)` | Cosine |
| `cosh(x)` | Hyperbolic cosine |
| `hypot(x,y)` | Hypotenuse |
| `sin(x)` | Sine |
| `sinh(x)` | Hyperbolic sine |
| `tan(x)` | Tangent |
| `tanh(x)` | Hyperbolic tangent |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/eval-functions
