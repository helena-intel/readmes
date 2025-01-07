# Benchmark vLLM model serving with OpenVINO backend


## Install

In short:


```
git clone https://github.com/vllm-project/vllm
cd vllm
python3 -m venv vllm_env
source vllm_env/bin/activate
pip install --upgrade pip
pip install -r requirements-build.txt --extra-index-url https://download.pytorch.org/whl/cpu
PIP_EXTRA_INDEX_URL="https://download.pytorch.org/whl/cpu" VLLM_TARGET_DEVICE=openvino python -m pip install -v .
```

See https://github.com/vllm-project/vllm/blob/main/docs/source/getting_started/installation/openvino.md for full instructions

## Serve

Set `VLLM_OPENVINO_KVCACHE_SPACE` to the amount of memory to use. 100 is recommended on a server with enough memory. See https://github.com/vllm-project/vllm/blob/main/docs/source/getting_started/installation/openvino.md about BKMs

```
VLLM_OPENVINO_KVCACHE_SPACE=100 VLLM_OPENVINO_CPU_KV_CACHE_PRECISION=u8 vllm serve /path/to/model
```

## Benchmark

### Download dataset

```
cd benchmarks
wget https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/resolve/main/ShareGPT_V3_unfiltered_cleaned_split.json
```

### Run benchmark

In a different terminal, after starting the model server with the command above, and after activating the virtual environment, run:

```
python benchmark_serving.py --model /path/to/model --dataset-path /path/to/ShareGPT_V3_unfiltered_cleaned_split.json --num-prompts 500
```

Adjust num-prompts to your preference. See `python benchmark_serving.py -h` for all arguments.

> [!NOTE]
> Make sure to use the exact same model path syntax for the `serve` and the `benchmark_serving.py` steps. Do not use `/home/user` in one and `$HOME` in the other, do not have one model end with a slash and not the other. The arguments should be exactly the same.
