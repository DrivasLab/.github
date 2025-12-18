# Contributing to DrivasLab Projects

These guidelines apply to all DrivasLab repositories unless a repo has its own `CONTRIBUTING.md` or `AGENTS.md` that overrides them.

## Philosophy

We're a biomedical research lab. Our code exists to answer scientific questions and support publications. These are the things we care about:

- **Correctness over cleverness** — Simple code that's obviously right beats clever code that's subtly wrong.
- **Reproducibility matters** — A reviewer (or future lab member) should be able to regenerate any result.
- **Documentation helps everyone** — Future you will thank present you.

None of this is strictly enforced—use your judgment. But if you're ever lost or unsure, these principles are a good guide.

## Best Practices

### Code Organization

```
project/
├── data/
│   ├── raw/           # Immutable inputs (never modify, never commit)
│   ├── interim/       # Intermediate outputs
│   └── processed/     # Final processed data
├── src/               # Library code (importable modules)
├── scripts/           # CLI entrypoints that call src/
├── notebooks/         # Exploration and visualization (keep small)
├── conf/              # Configuration files (Hydra, YAML, etc.)
├── tests/             # Test suite
├── docs/              # Documentation
├── results/           # Figures, tables, model outputs
└── README.md          # Always have one
```

### Configuration

**Prefer configuration over hardcoding:**

```python
# 🚫 Avoid
data_path = "/project/drivas_shared/datasets/pmbb/v3/"

# ✅ Better
data_path = config.paths.raw_data  # from Hydra, argparse, or config file
```

This makes code portable across users and machines.

### Data Safety

- **Never delete data automatically.** No `rm -rf` in scripts. No destructive operations without explicit user confirmation.
- **Treat raw data as immutable.** Process into `interim/` or `processed/`, never modify originals.
- **Don't commit large files.** Use `.gitignore` for data directories. Reference data by path in documentation.

### Documentation Standards

Every analysis or pipeline should produce:

1. **A README** explaining what it does and how to run it
2. **Reproducible outputs** — someone else should be able to run your code and get the same results
3. **Clear provenance** — where did the input data come from? What parameters were used?

For significant pipelines, consider adding:
- A manifest or metadata file (JSON) with parameters and file hashes
- A human-readable summary of outputs

### Environment Management

```bash
# Always use python -m to avoid PATH issues on the cluster
python -m pip install -e .     # NOT: pip install -e .
python -m pytest tests/ -v     # NOT: pytest tests/ -v
```

Use conda environments. Include an `environment.yml` or `requirements.txt`.

## Git Workflow

### Commits

Write meaningful commit messages:

```
# 🚫 Avoid
git commit -m "fix"
git commit -m "updates"
git commit -m "asdfasdf"

# ✅ Better  
git commit -m "Fix off-by-one error in phenotype alignment"
git commit -m "Add QC step for missing genotypes"
```

### Branches

For solo projects, working on `main` is fine. For collaborative work:

- `main` — stable, working code
- `dev` or feature branches — work in progress
- Use pull requests for significant changes

### What to Commit

**Do commit:**
- Source code (`src/`, `scripts/`)
- Configuration files (`conf/`)
- Documentation (`docs/`, `README.md`)
- Small notebooks with cleared outputs
- Test files
- `environment.yml` / `requirements.txt`

**Don't commit:**
- Data files (raw, interim, or processed)
- Large outputs (models, figures) — unless they're small and final
- Credentials, API keys, or paths specific to your machine
- `.pyc`, `__pycache__`, `.ipynb_checkpoints`

Use a `.gitignore`:

```gitignore
# Data
data/
*.csv
*.parquet
*.pkl
*.npy

# Outputs
results/
*.log
logs/

# Python
__pycache__/
*.pyc
.venv/
*.egg-info/

# Jupyter
.ipynb_checkpoints/

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
```

## Code Style

### Python

- Use type hints where helpful (especially function signatures)
- Prefer explicit over implicit
- Keep functions small and focused
- Use descriptive variable names (`phenotype_matrix` not `pm`)

We don't enforce a specific formatter, but if you use one, `black` and `ruff` are good choices.

### R

- Use tidyverse style where appropriate
- Comment liberally
- Use RMarkdown for analyses that produce reports

## Testing

Tests aren't always necessary for exploratory analysis, but for reusable code:

```bash
python -m pytest tests/ -v
```

At minimum, test that:
- Data loading works and produces expected shapes
- Key functions don't crash on typical inputs
- Edge cases are handled (empty inputs, missing values)

## Questions?

- **Technical questions:** Ask Yunjun in Slack, or post in the LPC support channel
- **Cluster/account issues:** PMACS help desk or LPC support Slack channel
- **Process questions:** Check the [Lab Manual](https://www.notion.so/pennibi/b3de144e483e40ec86fafd6b3d376ca5) or ask Teddy
