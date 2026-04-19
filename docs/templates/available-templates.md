# Available Templates

ExpOps ships with two built-in project templates. Use the `--template` flag with `expops create` to start from either one.

## sklearn-basic

A minimal, self-contained project for getting started with ExpOps. All ML logic lives in custom `@step`-decorated Python functions.

**Best for**: Learning the platform, experimenting with custom model code.

```bash
expops create my-project --template sklearn-basic
expops run my-project --local
```

**What's included**:

- `configs/project_config.yaml` — basic pipeline config with a local SQLite backend
- `src/models/model.py` — custom train/evaluate steps using `@step`
- `src/charts/plot_metrics.py` — a `@chart()` function for static PNG output
- `requirements.txt` — sklearn and standard dependencies
- `data/train.csv` — small example dataset

See the [sklearn-basic template guide](sklearn-basic.md) for a full walkthrough.

---

## sklearn-component

A template built around the [sklearn component library](../components/sklearn.md) and [matplotlib component library](../components/matplotlib.md). All ML logic is driven by `component:` keys in the config — no custom Python step code required.

**Best for**: Standard sklearn workflows, quickly trying different estimators, learning the component system.

```bash
expops create my-project --template sklearn-component
expops run my-project --local
```

**What's included**:

- `configs/project_config.yaml` — pipeline using `sklearn.*` and `matplotlib.*` components
- `src/plot_iterative_loss.js` — Chart.js dynamic chart for MLP training loss
- `requirements.txt` — `expops-sklearn` and `expops-matplotlib`
- `data/train.csv` — small example dataset

To swap the estimator, change the class name in the `component:` string and update `parameters` as needed:

```yaml
component: "sklearn.RandomForestClassifier.fit_predict"
parameters:
  n_estimators: 100
```

---

## premier-league

A comprehensive example project that demonstrates the full platform: distributed execution, dynamic JS charts, multiple environments, and a GCP backend.

**Best for**: Understanding advanced configuration, reference for production-style setups.

```bash
expops create my-project --template premier-league
```

**What's included**:

- Full cluster config (`configs/compute_config.yaml`) for SLURM/Dask execution
- Multiple pipeline environments
- Dynamic Chart.js visualisation (`src/plot_metrics.js`)
- GCP backend configuration (Firestore + GCS)

See the [Premier League template guide](premier-league.md) for details.
