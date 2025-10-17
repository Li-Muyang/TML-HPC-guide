# Installing Flash Attention on Setonix

This page will guide you on the installation process of vLLM on Setonix. 

## Prelimnary

Make sure the following before commence:
1. You read the main page on the usage of HPC.
2. You are now in a virtual environment with torch installed.
3. You have loaded correct version of ROCM (either 6.2.4 or 6.3.2 with matched Torch version).
4. You have installed correct PyTorch version with correct ROCM correspondence.

## Install

First make sure you have the correct ROCM and PyTorch version:

```bash
module load rocm/6.3.2
pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/rocm6.3
```

Also make sure you have followed the [previous guide](https://github.com/Li-Muyang/TML-HPC-guide/blob/main/advanced/install_flash_attn.md) and you have trion and flash attention installed.

Currently we only recommend compile and install from source, and only `vllm==0.9.1+rocm632` has been tested

```bash
git clone https://github.com/vllm-project/vllm.git && cd vllm
git checkout v0.9.1
pip install amdsmi==6.3.2
pip install --upgrade numba \
    scipy \
    huggingface-hub[cli,hf_transfer] \
    setuptools_scm
pip install "numpy<2"
pip install -r requirements/rocm.txt
export PYTORCH_ROCM_ARCH="gfx90a;gfx942"
module load gcc/12.2.0
CC=$(which gcc) CXX=$(which g++) MAX_JOBS=4 python setup.py develop
```

## Getting Help

For questions or issues, contact: **muyang.li@sydney.edu.au**


Make a pull request if you want to contribute to this guide.




