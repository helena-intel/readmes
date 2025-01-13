# OpenVINO GenAI LLM tips

## Inference best practices

For chat/instruct models, always use a system prompt and a chat template. See https://github.com/helena-intel/snippets/blob/main/llm_chat

## NPU

For NPU, please follow [this
guide](https://github.com/helena-intel/openvino/blob/patch-2/docs/articles_en/learn-openvino/llm_inference_guide/genai-guide-npu.rst)
for best practices. In particular: 
- Make sure that the NPU driver is updated ([Windows](https://www.intel.com/content/www/us/en/download/794734/intel-npu-driver-windows.html), [Linux](https://github.com/intel/linux-npu-driver/releases))
- Use symmetric INT4 quantization (`--weight-format int4 --sym`) to export the model
  - for models > 4GB, use `--group-size -1`
  - for smaller models start with `--group-size 128`
- Use OpenVINO 2024.6.
- Only greedy search is supported (use `do_sample=False` in `pipe.generate()`).
- Use model caching: set `{ "NPUW_CACHE_DIR": "model_cache" }` in `pipeline_config` and load the model with  `pipe = ov_genai.LLMPipeline(model_path, "NPU", **pipeline_config)`

Example `optimum-cli` command: 

```sh
optimum-cli export openvino -m meta-llama/Llama-2-7b-chat-hf --weight-format int4 --sym --group-size -1 --ratio 1.0 --awq --scale-estimation --dataset wikitext2 Llama-2-7b-chat-hf-ov-int4
```

Use `optimum-cli export openvino --help` to see all options.

## Optimum Intel

Update optimum-cli to the latest version with

```sh
pip install --upgrade --upgrade-strategy eager optimum-intel[openvino]
```

See this [installation
guide](https://github.com/helena-intel/optimum-intel/wiki/OpenVINO-Integration-Installation-Guide) for step by step
instructions and tips.

## INT4 weight quantization

The NNCF team has created recommended quantization configs for popular models which you can see here
https://github.com/huggingface/optimum-intel/blob/main/optimum/intel/openvino/configuration.py#L44 . If you use
optimum-cli with --weight-format int4 and no other weight compression options, this default config is applied. The
configs have been found to have a good balance of accuracy and performance. That doesn't mean they are the best for all
use cases, but they are a good starting point. Note that as mentioned above, for NPU some modifications are needed.
 
For models that do not have default configs (yet), it is usually helpful to use `--awq --dataset wikitext2` for better
accuracy. This documentation page explains the parameters to use. The page is about the API, but the same options are
supported in optimum-cli https://huggingface.co/docs/optimum/main/intel/openvino/optimization#4-bit

## Nightly

If you encounter issues with a particular model, it is often useful to try a nightly build:

```sh
pip install --pre --upgrade openvino-genai openvino openvino-tokenizers --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly
```

(there may also be issues with nightly, so this is not a recommendation to use nightly everywhere).

