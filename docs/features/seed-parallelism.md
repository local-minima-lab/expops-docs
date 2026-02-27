### Seed Parallelism

Seed parallelism duplicates the downstream process graph and runs each duplicate under a different random seed. It uses deterministic task-level seeding so that each duplicated branch is reproducible.

## Config

### Predefined seed split (no `code`)

```yaml
processes:
  - name: "seed_parallel"
    description: "Fan out by seed"
    seed_parallelism:
      seeds: [41, 42, 43]
```

- The split process has no `code`; it will merge upstream outputs and pass them through.
- The engine duplicates all downstream nodes until a `seed_aggregation` node is reached.
- Duplicated nodes are tracked via structured `seed_value` metadata on each process instance.

### User-defined seed split (`code`)

```yaml
processes:
  - name: "seed_parallel"
    description: "Custom seed hook"
    code: "define_seed_parallel_process"
    seed_parallelism:
      seeds: [41, 42, 43]
```

```python
from expops.core import process

@process()
def define_seed_parallel_process(seeds, **inputs):
    return dict(inputs)
```

- The `seeds` list is passed to the user-defined function as a kwarg.

## Aggregation

Add a dedicated aggregation process to collapse the **latest** seed-parallel layer:

```yaml
processes:
  - name: "aggregate_seeds"
    seed_aggregation: true
    code: "define_aggregate_seeds"
```

### Input shape at aggregation

When duplicate keys appear across seed branches, the aggregation process receives a dictionary keyed by seed:

```python
@process()
def define_aggregate_seeds(metrics):
    # metrics looks like: {"seed41": <val>, "seed42": <val>, ...}
    ...
```

When a node is **both** `data_aggregation` and `seed_aggregation`, the nested map is keyed by data first, then seed:

```python
@process()
def define_aggregate_data_and_seed(metrics):
    # metrics looks like: {"data1": {"seed41": <val>, ...}, "data2": {...}}
    ...
```

## Notes

- **UI representation**: Seed-parallel duplicates now use canonical XPath-style
  process IDs in the process graph (for example
  `//*[@partition='p1']/*[@seed='41']/process[@name='train']`), and human-readable
  labels such as `train P1 S41` are derived from this metadata.
- **Task-level RNG seeding**: Uses the most recent (innermost) seed layer when
  multiple seed-parallel layers are nested.
- **Aggregation behavior**: Each aggregation collapses only the most recent
  seed-parallel layer; outer layers remain as separate process nodes.
