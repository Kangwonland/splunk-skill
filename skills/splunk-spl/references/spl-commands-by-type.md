# SPL Commands by Distribution Type

Understanding command types is critical for performance optimization.

## Distributable Streaming
Run on individual indexers in parallel. Most efficient.

`abstract`, `addinfo`, `bin`/`bucket`, `convert`, `eval`, `extract`/`kv`, `fields`, `fieldformat`, `filldown`, `fillnull`, `head`, `kvform`, `lookup` (streaming), `makemv`, `multikv`, `nomv`, `regex`, `rename`, `replace`, `rex`, `rtorder`, `search`, `sort` (distributable), `spath`, `strcat`, `tail`, `where`, `xmlkv`, `xmlunescape`, `xpath`

## Centralized Streaming
Must run on search head. All data moves to search head first.

`anomalies`, `anomalydetection`, `anomalousvalue`, `autoregress`, `cluster`, `concurrency`, `delta`, `eventstats`, `foreach` (some modes), `iplocation`, `kmeans`, `makecontinuous`, `overlap`, `predict`, `rare` (some modes), `reltime`, `reverse`, `selfjoin`, `setfields`, `streamstats`, `transaction`, `trendline`, `uniq`, `x11`

## Transforming
Changes the structure of results. Runs after streaming commands.

`addcoltotals`, `addtotals`, `chart`, `contingency`, `stats`, `timechart`, `timewrap`, `top`, `transpose`, `untable`, `xyseries`

## Generating
Produces results independently (no input pipeline). Must be first command.

`dbinspect`, `eventcount`, `from` (SPL2), `gentimes`, `inputcsv`, `inputlookup`, `loadjob`, `makeresults`, `mcollect`, `metadata`, `mpreview`, `mstats`, `rest`, `savedsearch`, `search` (when first), `tstats`

## Orchestrating
Controls pipeline execution flow.

`append`, `appendcols`, `appendpipe`, `foreach`, `join`, `localop`, `map`, `redistribute`, `set`

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches
