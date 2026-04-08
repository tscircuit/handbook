# benchmark.sh

tscircuit has many algorithms for autorouting, placement, alignment, rendering, and more. It is often
difficult to know how to improve the performance or quality of an algorithm when working on a repository.
To make is easier to contribute meaningful algorithm fixes, tscircuit repos containing algorithms should
have a `benchmark.sh` file in the root matching the conventions in this document.

LLMs utilize `./benchmark.sh` to do "hillclimbing", where they incrementally improve an algorithm by trying
different strategies and rewrites. The `benchmark.sh` output should be helpful for LLMs to understand and
debug their solutions.

- `./benchmark.sh` should run a complete dataset by default and output all key metrics we want to improve
- `./benchmark.sh --help` should output meaningful/common usage and documentation for flags
- `./benchmark.sh --limit 20` should run a limited subset of a dataset
- `./benchmark.sh --sample NUM` should run a specific sample from the dataset
- Samples should output to `./results/run${number}/logs.txt` with additional snapshots in that directory
- Always include `duration=` in incremental log lines and summaries so that performance is tracked
- Use P50, P95 and avg as metrics. If you're not sure what to use, use P50 and P95.
- Metrics should have appropriate levels of accuracy, use `.toFixed(3)` or whatever rounding is most appropriate
- Pad incremental output such that the alignment is consistent

## Incremental Output

As `benchmark.sh` runs, it should output progress with key metrics in the output. Incremental progress allows
LLMs to stop the process early if they see obvious issues.

```
# ./benchmark.sh
sample001 success cm=100.0% drcIssues=07 lengthScore=0.983 duration=5.31s
# wrote ./results/run001/sample001/logs.txt ./results/run001/sample001/pcb.png
sample002 failed  cm=50.0%  drcIssues=12 lengthScore=10.0  duration=3.21s
# wrote ./results/run001/sample002/logs.txt ./results/run001/sample002/pcb.png
```

## Final Output

`benchmark.sh` should give an easy-to-parse summary.

```
success rate: 88.3%
avg DRC issues: 9
zero-DRC rate: 44%
P50 duration: 11.31s
P95 duration: 88.39s
```

## Recording baseline

To record a baseline, run `./benchmark.sh | tee baseline.txt`. If baselines are committed, make sure it is
performed automatically by CI on main branch merge. Generally not needed.
