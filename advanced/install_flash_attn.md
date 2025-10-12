# Installing Flash Attention on Setonix

This page will guide you on the installation process of flash attention. 

## Prelimnary

Make sure the following before commence:
1. You read the main page on the usage of HPC.
2. You are now in a virtual environment with torch installed.
3. You have loaded correct version of ROCM (either 6.2.4 or 6.3.2 with matched Torch version).

## Install

Currently we only recommend compile and install from source

```bash
git clone https://github.com/Dao-AILab/flash-attention.git && cd flash-attention
pip install triton==3.2.0
pip install wheel && pip install packaging
FLASH_ATTENTION_TRITON_AMD_ENABLE="TRUE" python setup.py install
```

Note that you have to set ```export FLASH_ATTENTION_TRITON_AMD_ENABLE="TRUE"``` all for new session you want to use flash attention, should either set this in your job script or make pernanment change by editing your bash script.

## Getting Help

For questions or issues, contact: **muyang.li@sydney.edu.au**


Make a pull request if you want to contribute to this guide.



