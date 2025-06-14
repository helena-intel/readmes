# Export Hugging Face models for OpenVINO NPU inference

This guide is for now focused on Windows, but the process is the same for Linux.

## Prerequisites

- Update NPU driver to latest version from https://www.intel.com/content/www/us/en/download/794734/intel-npu-driver-windows.html
- Install Git from https://git-scm.com/downloads/win and select the option to make git available outside of Git Bash.
- Python 3.9-3.12 from https://www.python.org/downloads/windows/
- OpenVINO version should be 2024.6 or later.

 
## Install Optimum Intel

Optimum Intel is used for model conversion/export.

It is recommended to [create a new virtual environment](https://github.com/helena-intel/readmes/blob/main/create_venv.md) to install the dependencies:

```
python -m venv venv
venv\Scripts\activate
pip install --upgrade pip
python -m pip install "optimum-intel[openvino]"@git+https://github.com/huggingface/optimum-intel.git
```

## Convert the model to OpenVINO. 

For NPU, it is important to export with "NPU friendly settings". These are: symmetric INT4 quantization (`--weight-format int4 --sym`) and for models larger than >4B channel-wise quantization (`--group-size -1`). For models smaller than <4B channel-wise quantization is not as important, but it does speed up inference and has a lower memory footprint. `--awq` is a setting to improve model output quality. It is not related to NPU and can be omitted.

In this example we use the model meta-llama/Llama-3.2-1B-Instruct . To use another model, replace that with the model_id from https://huggingface.co 

```
optimum-cli export openvino --sym --weight-format int4 --awq --dataset wikitext2 -m meta-llama/Llama-3.2-1B-Instruct Llama-3.2-1B-Instruct-ov-int4-sym
```

To export a model with more than 4B parameters, use channel-wise quantization with `--group-size -1`:

```
optimum-cli export openvino --sym --weight-format int4 --group-size -1 --awq --dataset wikitext2 -m meta-llama/Llama-3.1-8B-Instruct meta-llama/Llama-3.1-8B-Instruct-ov-int4-sym
```

See `optimum-cli export openvino --help` for all options. Using `--scale-estimation` could improve accuracy - but it takes quite a lot of time to export the model. 
Using `--all-layers` can improve performance, but can also reduce model output quality. For NPU-friendly models, do not change `--sym`, `--weight-format` and do not set `--ratio` to anything other than 1 (the default).

## Download llm_chat.py script

This script automatically sets a system prompt, and enables model caching for NPU.

```
curl -O https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/llm_chat/python/llm_chat.py
```

> [!NOTE]
> `curl` works differently in PowerShell. This `curl` command works in Command Prompt

## Run inference

```
python llm_chat.py Llama-3.2-1B-Instruct-ov-int4-sym NPU
```

The first time you run this it may take quite some time to compile the model to NPU. This will be much faster the next times, when the model will be loaded from cache.

### Tips

- Edit the script to change the system prompt to tweak model outputs
- In OpenVINO 2024.6, only greedy search is supported on NPU. Do not change `do_sample=False` in the script if you use 2024.6. Setting `temperature` and `top_p` has no effect. From OpenVINO 2025.0 sample search is supported on NPU too.
- By default the NPU pipeline supports an input size of up-to 1024 tokens. To increase that limit, for example to 2048, add `{"MAX_PROMPT_LEN": 2048}` to
`pipeline_config`. **Currently inputs up to about 4000 tokens are supported. Longer input sizes will be supported later this year**
- To speed up inference at the cost of slower model loading/compilation time, add `{"GENERATE_HINT": "BEST_PERF" }` to `pipeline_config`
- The llm_chat.py script works with chat models. These models usually have for example -instruct or -chat in their name. For non-chat models, or if you get an error about chat templates, try https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/llm_chat/python/llm_test.py instead. Do not expect chat quality from this - but it does allow you to test the model.

### C++ 

See [this sample](https://github.com/helena-intel/snippets/tree/main/llm_chat/cpp) for a C++ version of the chatbot.

### Troubleshooting

#### Cannot access gated repo

- If you try to export a "gated" model, for which you need to agree to terms to get access, you will get an error when running `optimum-cli`. Assuming you do have access to the model:
  - go to https://huggingface.co/settings/tokens and generate a token (can be a read token, or a fine-grained token with read access to the models you have access to)
  - type `huggingface-cli login` in the command prompt (with the environment where you run `optimum-cli` activated), and paste the token. The token will not be visible in the command prompt, so just press Enter after pasting. You will then be asked if the token should be added as a Git credential. Git credentials are not needed for this process, so you can answer `N` to that question.

You only need to run `huggingface-cli` once; your token will be saved. If you ever want to know if you are logged in, run `huggingface-cli whoami`. 

## Supported models

These models have been tested and are officially supported with NPU:

- meta-llama/Meta-Llama-3-8B-Instruct
- meta-llama/Llama-3.1-8B
- microsoft/Phi-3-mini-4k-instruct
- Qwen/Qwen2-7B
- mistralai/Mistral-7B-Instruct-v0.2
- openbmb/MiniCPM-1B-sft-bf16
- TinyLlama/TinyLlama-1.1B-Chat-v1.0
- TheBloke/Llama-2-7B-Chat-GPTQ
- Qwen/Qwen2-7B-Instruct-GPTQ-Int4

Similar models are expected to work too.
