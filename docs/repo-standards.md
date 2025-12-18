# Repository Standards

This guide describes different types of repos and suggestions for each. Nothing here is strictly enforced—use your judgment. The goal is just to keep things organized enough that others (and future you) can find and understand your work.

## The Three Types

### 1. Paper Repos

**Purpose:** Analysis code tied to a specific publication.

**Lifespan:** Active during project → archived after publication.

**Examples:** `ciliopathy-phewas-2024`, `cilia-twas-analysis`

**Good to have:**
- [ ] README with:
  - What this is (one paragraph)
  - How to reproduce key results
  - Link to paper/preprint (once available)
- [ ] `environment.yml` or `requirements.txt`
- [ ] Clear path from raw data → figures in paper
- [ ] Data availability statement (where to get the data, or note if restricted)

**Nice to have:**
- Numbered scripts matching figure numbers (`01_process_data.py`, `02_fig1_manhattan.py`)
- Snakemake/Nextflow workflow for full reproducibility

**After publication:** Archive the repo (make read-only). Future you will thank present you.

---

### 2. Tool/Pipeline Repos

**Purpose:** Reusable code meant to be run multiple times or by multiple people.

**Lifespan:** Long-lived, actively maintained.

**Examples:** `phewas-graph-ml`, `predixcan-pipeline`, `cilia-detection`

**Good to have:**
- [ ] Comprehensive README with:
  - Installation instructions
  - Quick start example
  - Configuration options
- [ ] `environment.yml` or `requirements.txt`
- [ ] Modular code structure (`src/` package, `scripts/` entrypoints)
- [ ] Configuration via files or CLI args (not hardcoded paths)
- [ ] Tests (`python -m pytest tests/`)

**Best practices:**
- `AGENTS.md` or `CONTRIBUTING.md` with project-specific conventions
- Hydra or similar for configuration management
- CI/CD (GitHub Actions) for automated testing
- Versioned releases for major milestones
- Documentation in `docs/` for complex projects

---

### 3. Scratch/Exploratory Repos

**Purpose:** Quick explorations, rotation projects, learning exercises.

**Lifespan:** Short. Either promoted to a real repo or deleted.

**Examples:** `jsmith-rotation-fall2024`, `explore-new-method`

**Good to have:**
- [ ] README with one sentence explaining what this is
- [ ] Your name and date somewhere

**Guidance:**
- Keep in your personal GitHub or a clearly-named subdirectory
- Don't expect others to maintain or understand this
- If it becomes useful, refactor into a paper or tool repo
- Delete when no longer needed (dead repos create confusion)

---

## Naming Conventions (Suggested)

We don't enforce strict naming, but consistency helps:

| Type | Pattern | Examples |
|------|---------|----------|
| Paper | `topic-year` or `descriptive-name` | `ciliopathy-phewas-2024` |
| Tool | `descriptive-name` | `phewas-graph-ml`, `predixcan-pipeline` |
| Scratch | `username-topic` or `explore-topic` | `yjk-rotation-2024`, `explore-cuml` |

Avoid:
- `analysis_v2_FINAL_really_final`
- `test`, `temp`, `stuff`
- Names that only make sense to you right now

---

## README Template

Every repo should have a README. Here's a minimal template:

```markdown
# Project Name

One-paragraph description of what this is and why it exists.

## Quick Start

\`\`\`bash
# Installation
conda env create -f environment.yml
conda activate myenv

# Run
python scripts/main.py --config conf/default.yaml
\`\`\`

## Data

- Input: Description of required input data and where to get it
- Output: What this produces and where it goes

## Project Structure

\`\`\`
├── data/           # Data directory (not committed)
├── src/            # Source code
├── scripts/        # CLI entrypoints
├── conf/           # Configuration
├── results/        # Outputs
└── README.md
\`\`\`

## Contact

Your Name (your.email@pennmedicine.upenn.edu)
```

---

## When to Create a New Repo

**Good reasons to create a new repo:**
- Starting a new paper/project
- Building a reusable tool
- The work is distinct enough to stand alone

**Maybe don't need a new repo:**
- It's a small script that belongs in an existing project
- You're just experimenting (use a branch or scratch repo)

**When in doubt:** Ask in Slack. Generally better to have fewer, organized repos than a bunch of abandoned ones.
