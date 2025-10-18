```bash
cd /work/xg25i074/x10515
module load python/3.10.16
module load cuda/12.8
module unload nvidia
module load gcc/12.4.0

. openr1_env/bin/activate
pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/cu128
https://github.com/vllm-project/vllm.git && cd vllm
git checkout v0.10.1
python use_existing_torch.py
pip install -r requirements/build.txt
CC=$(which gcc) CXX=$(which g++) MAX_JOBS=4 pip install --no-build-isolation -e .
```
