# Dry Lab Onboarding

Welcome to the computational side of Drivas Lab! This guide gets you from zero to running jobs on the cluster.

## Prerequisites

Before starting here, make sure you've completed the general lab onboarding in Notion:

- [ ] PennMedicine email address
- [ ] PMACS computing account
- [ ] Slack access (DrivasLab workspace)
- [ ] Notion access (Lab Manual)
- [ ] VPN client installed (GlobalProtect)

If you're missing any of these, see the [Onboarding page](https://www.notion.so/pennibi/b3de144e483e40ec86fafd6b3d376ca5) in Notion or ask Teddy.

## Day 1: Access

### 1. VPN Setup

You need VPN to access the cluster from off-campus.

1. Install **GlobalProtect** (not Ivanti—that's the old client)
2. Connect to `vpn.upenn.edu`
3. Authenticate with PennKey + DUO

Note: Penn uses a full-tunnel VPN, meaning all traffic routes through Penn while connected.

### 2. SSH to Prometheus

```bash
ssh YOUR_PENNKEY@prometheus.pmacs.upenn.edu
```

First login will prompt you to accept the host key. Say yes.

**Pro tip:** Set up SSH keys so you don't need a password every time. See the [Cluster Guide](cluster-guide.md#ssh-key-setup).

### 3. Find Your Bearings

```bash
# Your home directory
cd ~
pwd  # /home/YOUR_PENNKEY

# Lab shared directory
cd /project/drivas_shared
ls
```

Key locations:
- `/project/drivas_shared/datasets/` — shared datasets (treat as read-only)
- `/project/drivas_shared/projects/` — active projects
- `/project/drivas_shared/tools/` — shared tools and environments

## Day 2: Environment

### 1. Conda/Python Setup

You have a few options:

**Option A: PMACS modules** (simplest)
```bash
module load python/3.11
```

**Option B: Shared lab miniforge** (more flexibility)
```bash
# Add to ~/.bashrc
source /project/drivas_shared/miniforge/etc/profile.d/conda.sh
```

**Option C: Your own conda installation** (full control)

Install miniforge/miniconda in your home directory. Useful if you need specific configurations.

After setup:
```bash
source ~/.bashrc
conda activate base
```

### 2. Create a Project Environment

For a new project:

```bash
# Create environment from a file
conda env create -f environment.yml -n myproject

# Or create empty and add packages
conda create -n myproject python=3.11
conda activate myproject
python -m pip install pandas numpy  # Note: python -m pip, not bare pip
```

### 3. Editor Setup

**VS Code with Remote SSH** is a good option for working on the cluster:

1. Install [VS Code](https://code.visualstudio.com/) locally
2. Install the "Remote - SSH" extension
3. `Cmd/Ctrl + Shift + P` → "Remote-SSH: Connect to Host"
4. Connect to `prometheus.pmacs.upenn.edu`
5. Open folders, edit files, run terminals—all on the remote machine

**Optional extensions that make life easier:**
- **Python (Microsoft):** Syntax highlighting, autocomplete, debugging, Jupyter support
- **GitLens:** See who changed what and when, view file history

Use whatever editor you're comfortable with—vim, nano, PyCharm, whatever works.

## Day 3: Running Jobs

### Interactive Work

For quick tests and exploration:

```bash
# Start an interactive session
bsub -Is -q epistasis_interactive bash

# Now you're on a compute node, not the head node
# Do your work...

# Exit when done
exit
```

**Never run heavy computation directly on prometheus.** It's a shared head node.

### Batch Jobs

For real work, submit batch jobs:

```bash
# submit_job.sh
#!/bin/bash
#BSUB -J myjob
#BSUB -o logs/myjob.%J.out
#BSUB -e logs/myjob.%J.err
#BSUB -q epistasis_normal
#BSUB -n 1
#BSUB -M 16000

source /project/drivas_shared/miniforge/etc/profile.d/conda.sh
conda activate myproject

cd /project/drivas_shared/projects/myproject
python scripts/run_analysis.py
```

Submit:

```bash
mkdir -p logs
bsub < submit_job.sh
```

Monitor:

```bash
bjobs        # Your jobs
bjobs -u all # Everyone's jobs
bkill JOBID  # Kill a job
```

See the [Cluster Guide](cluster-guide.md) for more LSF details.

## Day 4: Git & GitHub

### 1. Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@pennmedicine.upenn.edu"
```

### 2. Clone a Repo

```bash
cd /project/drivas_shared/projects
git clone https://github.com/DrivasLab/REPONAME.git
cd REPONAME
```

### 3. Workflow

```bash
# Make changes
git add .
git commit -m "Add phenotype QC step"
git push
```

Read the [Contributing Guidelines](../CONTRIBUTING.md) for best practices.

## Project Structure

When starting a new analysis, use this structure:

```
myproject/
├── README.md          # What is this? How do I run it?
├── environment.yml    # Conda environment
├── conf/              # Configuration files
├── data/
│   ├── raw/           # Immutable inputs (symlink to shared data)
│   ├── interim/       # Intermediate files
│   └── processed/     # Final outputs
├── src/               # Reusable code
├── scripts/           # Entry points
├── notebooks/         # Exploration (keep small)
└── results/           # Figures, tables
```

**Key principle:** If you became unreachable, could someone else figure out where everything is?

## Getting Help

1. **Slack:** Ask Yunjun or post in the LPC support channel
2. **GitHub Issues:** For repo-specific problems
3. **PMACS Help Desk:** For cluster/account issues → helpdesk.pmacs.upenn.edu
4. **This documentation:** Check the [Cluster Guide](cluster-guide.md) and [Contributing Guidelines](../CONTRIBUTING.md)

## Next Steps

- [ ] SSH into prometheus successfully
- [ ] Activate conda and create a test environment
- [ ] Submit a test batch job
- [ ] Clone a lab repo
- [ ] Read the [Contributing Guidelines](../CONTRIBUTING.md)
- [ ] Explore an existing project to see how it's structured

Welcome to the lab! 🧬
