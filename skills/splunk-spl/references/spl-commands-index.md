# SPL Commands Index (Alphabetical)

All SPL commands with one-line descriptions.

| Command | Description |
|---------|-------------|
| `abstract` | Produce a summary of each search result |
| `accum` | Calculates the cumulative sum of a field |
| `addcoltotals` | Computes column totals for numeric fields |
| `addinfo` | Add information fields about the current search |
| `addtotals` | Computes the sum of all numeric fields per row |
| `analyzefields` | Analyze how well a field predicts another field |
| `anomalies` | Compute an anomaly score for search results |
| `anomalydetection` | Identify anomalous events using statistical models |
| `anomalousvalue` | Identify fields that contain unexpected values |
| `append` | Append results of a subsearch to current results |
| `appendcols` | Append field results from a subsearch to current results |
| `appendpipe` | Append results of the pipeline to current results |
| `arules` | Finds association rules between field values |
| `associate` | Identifies correlations between fields |
| `autoregress` | Sets up data for calculating the autoregression |
| `bin` / `bucket` | Puts continuous numerical values into discrete sets |
| `chart` | Returns results in a tabular format for charting |
| `cluster` | Clusters similar events together |
| `cofilter` | Filter out events based on co-occurrence |
| `collect` | Puts search results into a summary index |
| `concurrency` | Calculate concurrent events |
| `contingency` | Builds a contingency table for two fields |
| `convert` | Converts field values using various conversion functions |
| `correlate` | Calculates correlation between fields |
| `datamodel` | Returns the data model hierarchy |
| `datamodelsimple` | Returns a simplified data model |
| `dbinspect` | Returns information about Splunk indexes |
| `dedup` | Removes subsequent results that match a specified field list |
| `delete` | Deletes events from an index |
| `delta` | Computes the difference in field values between events |
| `diff` | Returns the difference between two search results |
| `erex` | Automatically identifies and extracts field values |
| `eval` | Calculates an expression and puts results in a field |
| `eventcount` | Returns the number of events in an index |
| `eventstats` | Adds summary statistics fields to search results |
| `expand` | Expand multivalue fields into multiple events (SPL2) |
| `extract` / `kv` | Extracts field-value pairs from _raw |
| `fieldformat` | Specify how field values should be displayed |
| `fields` | Removes fields from search results |
| `fieldsummary` | Returns summary statistics for all fields |
| `filldown` | Propagates the last non-null field value downward |
| `fillnull` | Replaces null values with a string |
| `findtypes` | Get suggested event types for a dataset |
| `folderize` | Create a folder structure in key-value data |
| `foreach` | Run a template pipeline over fields or values |
| `format` | Formats the results of a subsearch into a string |
| `from` | Retrieve data from a named dataset (SPL2) |
| `gauge` | Transforms results into a format for gauge charts |
| `gentimes` | Generates time-range results |
| `geom` | Add fields for geographic data visualizations |
| `geomfilter` | Remove events outside a bounding box |
| `geostats` | Aggregate geographical data for map charts |
| `head` | Returns the first N results |
| `highlight` | Highlights search terms in search results |
| `history` | Returns past searches from a search head |
| `iconify` | Display a unique icon for each different value |
| `inputcsv` | Loads search results from a CSV file |
| `inputlookup` | Loads results from a lookup table |
| `iplocation` | Extracts location information from IP addresses |
| `join` | Combine results of a subsearch with outer results |
| `kmeans` | Partitions search results into clusters |
| `kvform` | Extracts values from search results using a form |
| `loadjob` | Loads events from a previously completed search job |
| `localop` | Run subsequent commands on search head only |
| `lookup` | Add field values from an external source |
| `makecontinuous` | Makes a field that is a continuous time range |
| `makemv` | Turns a delimited string field into multivalue field |
| `makeresults` | Creates a specified number of empty results |
| `map` | Runs a search for each result |
| `mcollect` | Inserts data into a metrics index |
| `metadata` | Returns a list of sources, sourcetypes, or hosts |
| `metasearch` | Searches index metadata without retrieval |
| `mpreview` | Preview raw metric data from a metrics index |
| `mstats` | Calculate statistics from a metrics index |
| `multikv` | Extracts field-value pairs from table-formatted events |
| `multisearch` | Run multiple streaming searches simultaneously |
| `mvcombine` | Combines events with single/multivalue field differences |
| `mvexpand` | Expands the values of a multivalue field into separate events |
| `nomv` | Replaces multivalue field with single-value field |
| `outlier` | Removes or marks outliers in time-series data |
| `outputcsv` | Saves search results to a CSV file |
| `outputlookup` | Writes search results to a lookup file |
| `outputtext` | Outputs search results as plain text |
| `overlap` | Finds events with time overlaps |
| `pivot` | Run pivot searches against a particular data model |
| `predict` | Enables you to predict future values of a field |
| `rare` | Displays the least common values of a field |
| `redistribute` | Re-distribute events across indexers for parallel processing |
| `regex` | Remove results that do not match the specified regex |
| `reltime` | Converts the difference between _time and now to relative time |
| `rename` | Renames specified fields |
| `replace` | Replaces field values with a specified value |
| `require` | Specify required tags or eventtypes |
| `rest` | Access the Splunk REST endpoint |
| `return` | Specify the maximum number of values returned from a subsearch |
| `reverse` | Reverses the order of search results |
| `rex` | Extracts fields using a named group regular expression |
| `rtorder` | Buffers real-time events and orders them |
| `run` | Runs a custom function or script |
| `sample` | Returns a random sample of events |
| `savedsearch` | Returns results from a saved search |
| `search` | Filter results using search expressions |
| `searchtxn` | Finds the search string for a transaction |
| `selfjoin` | Joins results with itself on specified fields |
| `sendemail` | Sends an email with search results |
| `set` | Performs set operations on subsearch results |
| `setfields` | Sets field values |
| `sichart` | Summary index chart command |
| `sirare` | Summary index rare command |
| `sistats` | Summary index stats command |
| `sitimechart` | Summary index timechart command |
| `sitop` | Summary index top command |
| `sort` | Sorts search results by field |
| `spath` | Extracts information from XML and JSON formatted events |
| `stats` | Calculates aggregate statistics |
| `strcat` | Concatenates string values of two or more fields |
| `streamstats` | Calculates summary statistics for each event |
| `table` | Returns results in tabular format |
| `tags` | Displays tag information |
| `tail` | Returns the last N results |
| `timechart` | Per-time-bucket statistical aggregation for charting |
| `timewrap` | Displays series data over periods of time |
| `top` | Displays the most common values of a field |
| `transaction` | Groups events into transactions |
| `trendline` | Calculates moving averages of field values |
| `tscollect` | Puts search results into tsidx files |
| `tstats` | Calculates statistics over tsidx data |
| `typeahead` | Returns results from the typeahead index |
| `typelearner` | Generates suggested eventtypes |
| `typer` | Calculates the eventtype for each event (SPL2) |
| `union` | Merge results from multiple datasets (SPL2) |
| `uniq` | Removes duplicate events that are next to each other |
| `untable` | Converts tabular/columnar format to rows |
| `where` | Filters search results using eval expressions |
| `x11` | Produces the difference between seasonal and empirical data |
| `xmlkv` | Extracts XML key-value pairs |
| `xmlunescape` | Replaces XML escape sequences |
| `xpath` | Extracts information using XPath syntax |
| `xyseries` | Converts results into a format for chart visualizations |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4
