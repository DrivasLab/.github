# Prometheus / LPC Cluster Guide

This guide covers using Penn PMACS's LPC cluster, specifically our lab's nodes (`prometheus` and `epimetheus`).

## Overview

**What is LPC?**
- LPC (Limited Performance Computing) is a shared HPC resource managed by PMACS
- Jobs are scheduled via IBM LSF (job queuing system)
- We share queues (`epistasis_normal`, `epistasis_long`) with the Ritchie, Verma, and Setia-Verma labs

**Our resources:**
- Head node: `prometheus.pmacs.upenn.edu`
- Compute nodes: `prometheus`, `epimetheus`
- Shared storage: `/project/drivas_shared/`
- Interactive services: RStudio Workbench, Jupyter (on prometheus)

## Connecting

### VPN (Required for Off-Campus)

Penn uses **GlobalProtect** (not Ivanti/Pulse Secure—that's deprecated).

1. Download GlobalProtect from [Penn Software](https://www.software.upenn.edu/)
2. Connect to: `vpn.upenn.edu`
3. Authenticate with PennKey + DUO

⚠️ **Note:** This is a full-tunnel VPN. All your internet traffic routes through Penn while connected.

### SSH Access

```bash
ssh YOUR_PENNKEY@prometheus.pmacs.upenn.edu
```

### SSH Key Setup

Avoid typing your password every time:

**On your local machine:**

```bash
# Generate key pair (if you don't have one)
ssh-keygen -t ed25519 -C "your.email@pennmedicine.upenn.edu"

# Copy public key to cluster
ssh-copy-id YOUR_PENNKEY@prometheus.pmacs.upenn.edu
```

**SSH config** (`~/.ssh/config` on your local machine):

```
Host prometheus
    HostName prometheus.pmacs.upenn.edu
    User YOUR_PENNKEY
    IdentityFile ~/.ssh/id_ed25519
```

Now you can just:

```bash
ssh prometheus
```

### VS Code Setup

VS Code with Remote SSH is the recommended way to work:

1. Install [VS Code](https://code.visualstudio.com/)
2. Install extension: "Remote - SSH" (Microsoft)
3. `Cmd/Ctrl + Shift + P` → "Remote-SSH: Connect to Host"
4. Enter `prometheus` (if you set up SSH config) or full hostname
5. Open folders, edit files, run integrated terminal

**Optional but recommended extensions:**
- Python (Microsoft)
- Jupyter
- GitLens

### File Transfer

```bash
# Copy local → cluster
scp localfile.txt prometheus:/project/drivas_shared/projects/myproject/

# Copy cluster → local
scp prometheus:/project/drivas_shared/projects/myproject/results.csv ./

# Sync directories (recommended for larger transfers)
rsync -avz ./local_folder/ prometheus:/project/drivas_shared/projects/myproject/
```

For GUI-based transfer, use [Cyberduck](https://cyberduck.io/) (Mac) or [WinSCP](https://winscp.net/) (Windows).

## File System

### Key Directories

| Path | Purpose | Quota | Backup |
|------|---------|-------|--------|
| `/home/YOUR_PENNKEY/` | Personal home | Limited | Yes |
| `/project/drivas_shared/` | Lab shared space | Large | No |
| `/project/drivas_shared/datasets/` | Shared datasets | — | Treat as immutable |
| `/project/drivas_shared/projects/` | Active projects | — | — |

### Lab Directory Structure

```
/project/drivas_shared/
├── datasets/           # Shared data (read-only in practice)
│   └── pmbb/           # PennMedicine BioBank
├── projects/           # Active work
│   ├── phewas-graph-ml/
│   └── cilia-twas/
├── tools/              # Shared tools
└── miniforge/          # Shared conda installation
```

### Storage Best Practices

- **Don't fill your home directory** — use `/project/drivas_shared/` for project data
- **Don't commit data to git** — keep data in `data/` directories, add to `.gitignore`
- **Clean up intermediate files** — disk space is shared

Check usage:

```bash
# Your home directory
du -sh ~

# Lab directory
du -sh /project/drivas_shared/projects/*/
```

## LSF Job Scheduling

### Queues

| Queue | Max Runtime | Use Case |
|-------|-------------|----------|
| `epistasis_normal` | 24 hours | Most jobs |
| `epistasis_long` | 7 days | Long-running jobs |
| `epistasis_interactive` | 8 hours | Interactive work |

### Interactive Sessions

**Never run compute-intensive work directly on the head node.**

```bash
# Start interactive session
bsub -Is -q epistasis_interactive bash

# With more resources
bsub -Is -q epistasis_interactive -n 4 -M 32000 bash

# You're now on a compute node
# Do your work...
exit
```

### Batch Jobs

Create a submission script:

```bash
#!/bin/bash
#BSUB -J myjob                    # Job name
#BSUB -o logs/myjob.%J.out        # stdout (%J = job ID)
#BSUB -e logs/myjob.%J.err        # stderr
#BSUB -q epistasis_normal         # Queue
#BSUB -n 1                        # Number of cores
#BSUB -M 16000                    # Memory in MB
#BSUB -W 4:00                     # Wall time (HH:MM)

# Setup environment
source /project/drivas_shared/miniforge/etc/profile.d/conda.sh
conda activate myenv

# Run
cd /project/drivas_shared/projects/myproject
python scripts/analysis.py --config conf/run1.yaml
```

Submit:

```bash
mkdir -p logs
bsub < submit.sh
```

### Job Arrays

For parameter sweeps:

```bash
#!/bin/bash
#BSUB -J myjob[1-100]             # Array of 100 jobs
#BSUB -o logs/myjob.%J.%I.out     # %I = array index
#BSUB -e logs/myjob.%J.%I.err

python scripts/analysis.py --index $LSB_JOBINDEX
```

### Job Management

```bash
bjobs                    # Your jobs
bjobs -u all             # All users' jobs
bjobs -l JOBID           # Detailed job info
bkill JOBID              # Kill a job
bkill 0                  # Kill all your jobs
bqueues                  # Queue status
bhist -l JOBID           # Job history (after completion)
```

### Common BSUB Options

| Option | Description | Example |
|--------|-------------|---------|
| `-J` | Job name | `-J myanalysis` |
| `-o` | stdout file | `-o logs/%J.out` |
| `-e` | stderr file | `-e logs/%J.err` |
| `-q` | Queue | `-q epistasis_normal` |
| `-n` | CPU cores | `-n 4` |
| `-M` | Memory (MB) | `-M 32000` |
| `-W` | Wall time | `-W 24:00` |
| `-R` | Resource requirements | `-R "rusage[mem=8000]"` |

## Environment Management

### Conda

A few options for Python environment management:

**Option A: PMACS modules**
```bash
module load python/3.11
```

**Option B: Shared lab miniforge**
```bash
# Add to ~/.bashrc
source /project/drivas_shared/miniforge/etc/profile.d/conda.sh

# Then activate
conda activate myenv
```

**Option C: Your own installation**

Install miniforge/miniconda in your home directory if you want full control.

### Creating environments

```bash
# Create new environment
conda create -n newenv python=3.11
conda activate newenv
```

### Installing Packages

```bash
# Always use python -m pip (cluster has PATH issues with bare pip)
python -m pip install pandas numpy scikit-learn

# For a requirements file
python -m pip install -r requirements.txt
```

### Modules (PMACS-managed software)

PMACS provides some pre-installed software via modules:

```bash
module avail              # List available
module load python/3.9    # Load a module
module list               # Show loaded
module unload python/3.9  # Unload
```

Generally, prefer conda environments over modules for reproducibility.

## Interactive Services

### RStudio Workbench

1. Connect to VPN
2. Navigate to: `https://prometheus.pmacs.upenn.edu/`
3. Login with PMACS credentials
4. Select RStudio

### Jupyter

Same URL, select Jupyter instead of RStudio.

⚠️ These require licensed seats—ask Teddy if you need access.

## Troubleshooting

### "Permission denied" on SSH

```bash
# Check your key permissions (local machine)
chmod 600 ~/.ssh/id_ed25519
chmod 700 ~/.ssh
```

### Job stuck in PEND

```bash
bjobs -l JOBID  # Check why
```

Common reasons:
- Queue is full (wait)
- Requesting more resources than available
- Dependency on another job

### "Module not found" in batch job

Your batch job runs in a clean environment. Make sure to:
1. Source conda in the script
2. Activate your environment
3. Use full paths or `cd` to your project directory

### Out of memory

Your job got killed for exceeding memory. Solutions:
- Request more memory: `-M 64000`
- Process data in chunks
- Use memory-efficient data types (float32 vs float64)

### Disk quota exceeded

```bash
# Check usage
du -sh ~/*
du -sh /project/drivas_shared/projects/YOUR_PROJECT/*

# Clean up
rm -rf __pycache__/ .ipynb_checkpoints/
# Remove old logs, intermediate files
```

## Quick Reference

### Common Commands

```bash
# Connection
ssh prometheus
scp file.txt prometheus:/path/

# Jobs
bsub < script.sh          # Submit
bjobs                      # Status  
bkill JOBID                # Kill
bqueues                    # Queue info

# Environment
conda activate myenv
python -m pip install pkg
module avail

# Files
du -sh directory/          # Size
ls -la                     # List with details
```

### Useful Aliases

Add to `~/.bashrc`:

```bash
alias ll='ls -la'
alias jobs='bjobs'
alias sq='bqueues'

# Safety nets
alias rm='rm -i'
alias mv='mv -i'
alias cp='cp -i'
```

## Resources

- [PMACS LPC Wiki](https://wiki.pmacs.upenn.edu/pub/LPC)
- [LSF Documentation](https://www.ibm.com/docs/en/spectrum-lsf)
- [PMACS Help Desk](https://helpdesk.pmacs.upenn.edu/)

For cluster account issues, email: `pmacs-sys-sci@lists.upenn.edu`
