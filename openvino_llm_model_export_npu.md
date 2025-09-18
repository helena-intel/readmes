# Export Hugging Face LLM/VLMs for OpenVINO NPU inference

This guide is for now focused on Windows, but the process is the same for Linux.

## Prerequisites

- Update NPU driver to latest version from https://www.intel.com/content/www/us/en/download/794734/intel-npu-driver-windows.html (this is very important)
- Install Git from https://git-scm.com/downloads/win and select the option to make git available outside of Git Bash
- Python 3.9-3.12 from https://www.python.org/downloads/windows/
 
## Install Optimum Intel

Optimum Intel is used for model conversion/export.

It is recommended to [create a new virtual environment](https://github.com/helena-intel/readmes/blob/main/create_venv.md) to install the dependencies:

```
python -m venv venv
venv\Scripts\activate
pip install --upgrade pip
python -m pip install "optimum-intel[openvino]"@git+https://github.com/huggingface/optimum-intel.git
```

If you install in an existing environment, make sure that your OpenVINO version is at least 2025.1.

## Convert the model to OpenVINO. 

For NPU, it is important to export with "NPU friendly settings". These are: symmetric INT4 quantization (`--weight-format int4 --sym`) and for models larger than >4B channel-wise quantization (`--group-size -1`). For models smaller than <4B channel-wise quantization is not required, but it does speed up inference and model loading and has a lower memory footprint. `--awq` is a setting to improve model output quality. It is not related to NPU and can be omitted.

In this example we use the model meta-llama/Llama-3.1-8B-Instruct. To use another model, replace that with the model_id from https://huggingface.co 

```
optimum-cli export openvino --sym --weight-format int4 --group-size -1 --awq --m meta-llama/Llama-3.1-8B-Instruct Llama-3.1-8B-Instruct-ov-int4-sym
```

See `optimum-cli export openvino --help` for all options. Using `--scale-estimation` could improve accuracy - but it can take quite a lot of time to export the model. 
Using `--all-layers` can improve performance, but often reduces model output quality significantly. For NPU-friendly models, do not change `--sym`, `--weight-format` and do not set `--ratio` to anything other than 1 (the default).

## Download llm_chat.py script

This script enables model caching for NPU. Loading the model the first time will be slow, but subsequent times will be much faster.

> [!NOTE]
> There is an issue with CACHE_DIR with OpenVINO 2025.3. This is fixed in nightly: `pip install --pre -U openvino openvino-tokenizers openvino-genai --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly`

```
curl.exe -O https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/llm_chat/python/llm_chat.py
```

## Run inference

```
python llm_chat.py Llama-3.1-8B-Instruct-ov-int4-sym NPU
```

### Tips

- By default the NPU pipeline supports an input size of up-to 1024 tokens. To increase that limit, for example to 2048, add `{"MAX_PROMPT_LEN": 2048}` to
`pipeline_config`. **Currently inputs up to about 4000 tokens are supported. Longer input sizes will be supported later this year**
- The llm_chat.py script works with chat models. These models usually have for example -instruct or -chat in their name. For non-chat models, or if you get an error about chat templates, try https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/llm_chat/python/llm_test.py instead. Do not expect chat quality from this - but it does allow you to test the model.

### C++ 

See [this sample](https://github.com/helena-intel/snippets/tree/main/llm_chat/cpp) for a C++ version of the chatbot.

### Troubleshooting

#### Cannot access gated repo

- If you try to export a "gated" model, for which you need to agree to terms to get access, you will get an error when running `optimum-cli`. Assuming you do have access to the model:
  - go to https://huggingface.co/settings/tokens and generate a token (can be a read token, or a fine-grained token with read access to the models you have access to)
  - type `huggingface-cli login` in the command prompt (with the environment where you run `optimum-cli` activated), and paste the token. The token will not be visible in the command prompt, so just press Enter after pasting. You will then be asked if the token should be added as a Git credential. Git credentials are not needed for this process, so you can answer `N` to that question.

You only need to run `huggingface-cli login` once; your token will be saved. If you ever want to know if you are logged in, run `huggingface-cli whoami`. 

## Supported models

These LLMs have been tested and are officially supported with NPU:

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

For VLMs, these models are expected to work as of 2025.3:

- MiniCPM-V-2_6
- Phi-3.5-vision-instruct
- Phi-4-multimodal-instruct
- Qwen2.5-VL-3B-Instruct
- Qwen2.5-VL-7B-Instruct
