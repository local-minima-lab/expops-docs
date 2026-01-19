# Data and Seed Parallelism

ExpOps can parallelize pipeline execution by duplicating parts of the process graph across data partitions or random seeds. Both features are configured per process in `configs/project_config.yaml` under `model.parameters.pipeline.processes`.

## Data Parallelism

Use data parallelism when a process returns a dataframe that should be split and processed in parallel by downstream processes.

**Configuration keys**:
- `data_parallelism.size`: Number of partitions (int) or explicit row counts (list of ints).
- `data_parallelism.data_name`: Key in the process return dict that contains the dataframe.

**Behavior**:
- The dataframe is split into partitions.
- Downstream processes are duplicated per partition until a data aggregation boundary.
- Partitioned processes are suffixed with `_1`, `_2`, ... in the expanded graph.

**Example**:
```yaml
processes:
  - name: "feature_engineering"
    code_function: "define_feature_engineering_process"
    data_parallelism:
      size: 3 # or [50,20,20] indicating three partitions with a 50, 20, 20 row count split
      data_name: df
```

**Process return requirements**:
- The process must return a dict containing `data_name`.
- The value can be a pandas DataFrame, a list of records, or a dict (it will be converted).

## Seed Parallelism

Use seed parallelism to run downstream branches multiple times with deterministic seeds.

**Configuration keys**:
- `seed_parallelism.seeds`: List of integer seeds (can also be a single int or list in YAML).

**Behavior**:
- Downstream processes are duplicated once per seed until a seed aggregation boundary.
- Seeded processes are suffixed with `_seed{value}` in the expanded graph.
- The seed value is injected into the process hyperparameters as `random_seed`.

**Note**: ExpOps seeds common RNGs (Python, NumPy, PyTorch, TensorFlow) at task level, but it does not automatically set `random_state`/`seed` arguments for every library or model. If your library requires an explicit seed parameter, pass `hyperparameters["random_seed"]` in your process code.

**Example**:
```yaml
processes:
  - name: "train_model"
    code_function: "define_training_process"
    seed_parallelism:
      seeds: [21, 22, 23]
```

## Aggregation Boundaries

To stop graph duplication and combine results, mark a process as an aggregation boundary:

```yaml
processes:
  - name: "feature_engineering_generic"
    data_parallelism:
      size: 2
      data_name: df

  - name: "preprocess_linear_nn"
    seed_parallelism:
      seeds: [40, 41]

  - name: "ensemble_aggregation"
    data_parallelism:
      aggregation: true
    seed_parallelism:
      aggregation: true
```

Aggregation processes receive a merged `data` payload containing the partition/seed results keyed by the base process name.

