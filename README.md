# Setonix Usage Guide

This page is for **TML members** who have HPC needs and want to access resources through Setonix at Pawsey. This is a trimmed guide for usage, where the full guide can be found [here](https://pawsey.atlassian.net/wiki/spaces/US/pages/51925434/Setonix+User+Guide). We strongly recommend you to read the complete guide after reading this trimmed guide.

## Getting Access

Users who do not have an account should first discuss with Prof. Liu for approval, then send the following information to muyang.li@sydney.edu.au:

1. Full name
2. University email address
3. Estimated tasks that need to run and estimated job cost

## Terms of Use

Once you have an account, you must **strictly adhere** to the following rules before using the cluster:

1. Strictly comply with Setonix's [Terms of Use](https://pawsey.atlassian.net/wiki/spaces/US/pages/51926766/Conditions+of+Use)
2. Do not waste or over-use computational resources
3. Do not run code that does not contribute to TML projects

**Violation of these rules may result in loss of HPC privileges and other penalties.**

---

## 1. Working Directory Setup

By default, the HOME directory in Setonix is:

```bash
/home/username
```

However, this directory has strict I/O and space limits, so you **cannot run anything** under this directory.

Instead, everything must be placed under the `scratch` directory.

**The first thing everyone should do is:**

```bash
export HOME=/scratch/pawsey1015/username
```

This properly sets up the HOME directory. **Note:** This change to the environment variable will **disappear** once the current session ends, so apply this setup every time in a new session or make it permanent by adding it to your `.bashrc` file.

### Setting Cache Directories

Many packages such as PyTorch and Hugging Face have their own default cache directories that sometimes override HOME. Here is a common setup:

```bash
export HF_HOME="/scratch/pawsey1015/username/.cache/huggingface"
export HF_DATASETS_CACHE="/scratch/pawsey1015/username/.cache/huggingface/datasets"
export TRANSFORMERS_CACHE="/scratch/pawsey1015/username/.cache/huggingface/models"
export TORCH_HOME="/scratch/pawsey1015/username/.cache/torch/hub"
export HOME="/scratch/pawsey1015/username"
```

> **Key Takeaway:** Nothing should be saved under the `/home` directory. Always keep your working directory under `/scratch/pawsey1015/username`.

---

## 2. Setting Up Virtual Environment

For those with environment control needs, the recommended approach is using Python's built-in `venv` manager.

### Check Available Python Versions

First, check what Python versions are currently supported on Setonix:

```bash
module avail python
```

If you don't find your desired Python version, try:

```bash
module use /software/setonix/unsupported
module avail python
```

This allows you to check available but not yet officially supported software.

### Load Python and Create Virtual Environment

Suppose you want to work with Python 3.11.6:

```bash
module load python/3.11.6
```

Verify the loaded version:

```bash
which python && python -V
```

You should see the returned version matches your loaded version.

Now, create your virtual environment under the working directory (`/scratch/pawsey1015/username`):

```bash
python -m venv my_env
source my_env/bin/activate
```

You should now be working within your virtual environment.

> **Important:** Make sure the Python version you loaded matches the Python version you used when creating the virtual environment!

---

## 3. Installing PyTorch and Setting Up ROCm

ROCm is AMD's counterpart to CUDA. Everything you used to run with CUDA needs to run with ROCm on Setonix. Currently, most popular deep learning workflows are supported in ROCm.

### Check Available ROCm Versions

Similar to Python, check available ROCm versions:

```bash
module avail rocm
```

We currently recommend using **ROCm 6.2.4** or **ROCm 6.3.2**, depending on the PyTorch version you need to use.

### Check PyTorch and ROCm Compatibility

You can check the correspondence between PyTorch versions and ROCm versions at:  
https://pytorch.org/get-started/previous-versions/

### Install PyTorch

For example, if you want to use ROCm 6.2.4 with PyTorch 2.6.0:

```bash
module load rocm/6.2.4
pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/rocm6.2
```

> **Always ensure your PyTorch version and ROCm version are compatible!**

Most packages that are not GPU-dependent, such as NumPy, can be installed via standard `pip install`.

---

## 4. Debugging Code and Interactive Job Submission

Now that we have successfully set up the working directory and environments, we can start running code on GPUs.

### Request an Interactive GPU Session

For debugging, you can request an interactive GPU session:

```bash
salloc -p gpu-dev -N 1 --gres=gpu:1 -A pawsey1015-gpu --time=04:00:00
```

**Parameter Explanations:**

| Parameter | Description |
|-----------|-------------|
| `-p` | Partition to use (use `gpu-dev` for debugging) |
| `-N` | Number of nodes (change to 2 for multi-node applications) |
| `--gres=gpu:` | Number of GPUs per node (min: 1, max: 8 per node) |
| `-A` | Project affiliation (default: `pawsey1015-gpu`) |
| `--time` | Wall time requested (max: 4 hours for `gpu-dev`) |

> **Do not request more resources than you need!**

### Verify GPU Access

After running the above command, you will be in an interactive GPU session. Run:

```bash
rocm-smi
```

You should see the GPU you requested. Since you are now in a new session, you need to **repeat the process from sections 1 & 2** (export environment variables and activate your virtual environment).

> **Note:** Each user can only have **one interactive session**, and it is not meant for long-running jobs.

---

## 5. Submitting Jobs

Most computation should be done via SLURM job submission. You will need to create a job script and submit it.

### Example Job Script

Here is a working example (save as `dpo.sh`):

```bash
#!/bin/bash --login

#SBATCH --account=pawsey1015-gpu
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --gres=gpu:1
#SBATCH --time=24:00:00

# GPU support
export MPICH_GPU_SUPPORT_ENABLED=1
export OMP_NUM_THREADS=1

# Cache directories
export HF_HOME="/scratch/pawsey1015/username/.cache/huggingface"
export HF_DATASETS_CACHE="/scratch/pawsey1015/username/.cache/huggingface/datasets"
export TRANSFORMERS_CACHE="/scratch/pawsey1015/username/.cache/huggingface/models"
export TORCH_HOME="/scratch/pawsey1015/username/.cache/torch/hub"
export HOME="/scratch/pawsey1015/username"
export FLASH_ATTENTION_TRITON_AMD_ENABLE="TRUE"

# Load modules
module load python/3.11.6
module use /software/setonix/unsupported
module load rocm/6.2.4

# Navigate to project directory
cd /scratch/pawsey1015/username/project_dir

# Activate virtual environment
source /scratch/pawsey1015/username/my_env/bin/activate

# Disable wandb if not needed
export WANDB_MODE=disabled

# Run your script
python -u train.py
```

This script contains everything from steps 1-4. You can modify the variables according to your needs.

> **Note:** The longest wall time you can request for the `gpu` partition is **24 hours**.

### Submit the Job

Submit your job script:

```bash
sbatch dpo.sh
```

### Monitor Job Status

Check the status of your job:

```bash
squeue -u $USER
```

Or use:

```bash
sacct
```

**Job Status Meanings:**
- `PENDING`: Waiting to be executed
- `RUNNING`: Currently running
- `COMPLETED`: Successfully finished
- `FAILED`: Job failed

### Cancel a Job

If you no longer want to run a job:

```bash
scancel <job_id>
```

---

## Getting Help

For questions or issues, contact: **muyang.li@sydney.edu.au**


Make a pull request if you want to contribute to this guide.


