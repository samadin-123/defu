# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: pnpm build

## Setup

Install dependencies and prepare the evaluation environment:

```bash
# Install pnpm if not available
npm install -g pnpm@10.33.0

# Install project dependencies
pnpm install

# Build the project (TypeScript → JavaScript)
pnpm build

# Verify tests still pass (correctness check)
pnpm test
```

The `prereq_command` above ensures the benchmark runs against freshly compiled output after any source changes.

## Run command

```bash
node bench.mjs
```

The benchmark measures operations per second across four representative workloads:
- **Simple objects**: shallow property merging
- **Nested objects**: deep recursive merging
- **Arrays**: array concatenation performance
- **Complex objects**: mixed nested objects and arrays (real-world scenarios)

Results are weighted (complex scenarios count more) to reflect typical usage patterns.

## Output format

The benchmark must print `METRIC=<number>` to stdout.

Example output:
```
=== Benchmark Results ===
simple: 1,234,567 ops/sec
nested: 456,789 ops/sec
arrays: 567,890 ops/sec
complex: 345,678 ops/sec

Weighted average: 456,789 ops/sec
METRIC=456789
```

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Ground truth

The baseline metric represents the weighted average operations per second of the defu object merging algorithm across four test scenarios. The benchmark runs 100,000 iterations per scenario (after 10,000 warmup iterations) to minimize noise. The metric combines performance across simple, nested, array, and complex object merging with weights [1, 2, 2, 3] respectively to emphasize real-world usage patterns.

## Correctness validation

All changes must pass the existing test suite (`pnpm test`) which includes:
- Unit tests for defu, defuFn, and defuArrayFn
- Type checking with TypeScript
- Edge cases (null handling, array concatenation, nested merging, prototype pollution prevention)

The test suite serves as the correctness boundary — performance improvements that break tests are invalid.
