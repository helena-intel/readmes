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

See https://github.com/vllm-project/vllm/blob/main/docs/source/getting_started/installation/ai_accelerator/openvino.inc.md for full instructions

## Serve

Set `VLLM_OPENVINO_KVCACHE_SPACE` to the amount of memory to use. 100 is recommended on a server with enough memory. See https://github.com/vllm-project/vllm/blob/main/docs/source/getting_started/installation/ai_accelerator/openvino.inc.md about BKMs

`/path/to/model` in the examples below can refer to a path to an OpenVINO LLM, converted with optimum-intel ([documentation](https://huggingface.co/docs/optimum/main/en/intel/openvino/export)) or a model_id from the Hugging Face hub. If the model from the hub is not yet in OpenVINO format it will be converted on-the-fly, if the model is in OpenVINO format, it will be used as is.

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

## Test Inference

To run one inference on the model served by vLLM, one option is to use the OpenAI API (`pip install openai`):

```python
from openai import OpenAI

openai_api_key = "vllm-abc123456"
openai_api_base = "http://127.0.0.1:8000/v1"
client = OpenAI(api_key=openai_api_key, base_url=openai_api_base)

chat_response = client.chat.completions.create(
model = /path/to/model,
messages=[
  {"role": "system", "content": "You are a helpful assistant."},
  {"role": "user", "content": "Tell me a joke."},
]
).choices[0].message.content
print("Chat response:", chat_response)
```
