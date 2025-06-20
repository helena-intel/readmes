# OpenVINO GenAI LLM tips

## Inference best practices

- For chat/instruct models, starting with OpenVINO GenAI 2025.1, a chat template is enabled by default when running inference with `pipe.generate(prompt, config)`, even without `pipe.start_chat()`. 
To disable this, set `config.apply_chat_template = False`.
- For very basic sample code in C++ and Python, see https://github.com/helena-intel/snippets/blob/main/llm_chat
- Check out the [OpenVINO GenAI documentation](https://docs.openvino.ai/2025/openvino-workflow-generative/inference-with-genai.html) for more information, including streaming output and speculative decoding, and [OpenVINO GenAI samples](https://github.com/openvinotoolkit/openvino.genai/tree/master/samples/python/text_generation).

## Optimum Intel

Optimum Intel is a collaboration of Hugging Face and Intel to accelerate transformers models on Intel hardware. With OpenVINO GenAI, we use Optimum Intel to export transformer models
from the Hugging Face Hub to OpenVINO, using `optimum-cli`.

Install optimum-intel or upgrade to the latest version with:

```sh
pip install --upgrade --upgrade-strategy eager optimum-intel[openvino]
```

To get the version with the latest bugfixes and improvements, install optimum-intel from source with this command (this requires that [Git](https://git-scm.com/downloads) is installed):

```sh
pip install --upgrade --upgrade-strategy eager "optimum-intel[openvino]"@git+https://github.com/huggingface/optimum-intel.git
```

See this [installation
guide](https://github.com/helena-intel/optimum-intel/wiki/OpenVINO-Integration-Installation-Guide) for step by step
instructions and tips.

Optimum is only used for exporting models. It is not needed for running inference.

## INT4 weight quantization

The NNCF team has created recommended quantization configs for popular models which you can see here
https://github.com/huggingface/optimum-intel/blob/main/optimum/intel/openvino/configuration.py#L44 . If you use
`optimum-cli` with `--weight-format int4` and no other weight compression options, this default config is applied. The
configs have been found to have a good balance of accuracy and performance. That doesn't mean they are the best for all
use cases, but they are a good starting point. Note that for NPU some modifications are needed (see the NPU section below).

This documentation page explains the parameters to use for INT4 weight compression: https://docs.openvino.ai/2025/openvino-workflow/model-optimization-guide/weight-compression/4-bit-weight-quantization.html
It is a generic page about weight quantization options, `optimum-cli export openvino --help` shows the exact options to use with `optimum-cli`.
For models that do not have default configs (yet), it is often helpful to use `--awq --dataset wikitext2` for better accuracy.

## Nightly

If you encounter issues with a particular model, it is often useful to try a nightly build:

```sh
pip install --pre --upgrade openvino-genai openvino openvino-tokenizers --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly
```

(there may also be issues with nightly, so this is not a recommendation to use nightly everywhere).

## NPU

For NPU, please refer to the [GenAI NPU documentation](https://docs.openvino.ai/2025/openvino-workflow-generative/inference-with-genai/inference-with-genai-on-npu.html). Most importantly:
- Make sure that the NPU driver is updated ([Windows](https://www.intel.com/content/www/us/en/download/794734/intel-npu-driver-windows.html), [Linux](https://github.com/intel/linux-npu-driver/releases))
- Use symmetric INT4 quantization (`--weight-format int4 --sym`) to export the model
  - for models > 4GB, use channel-wise quantization: `--group-size -1`. Group-size quantization may work, but model loading time will be very slow.
  - for smaller models, both channel-wise (`--group-size -1`) and group-wise (`--group-size 128`) quantization is supported. Channel-wise quantization generally results in faster inference, faster model loading time, and lower memory use, so it is a good method to start with. Try group-wise quantization if accuracy with channel-wise quantization is not acceptable (for accuracy also see the note about `--awq` above). 
- Use OpenVINO GenAI 2025.0 or later.
- Use model caching: set `{"CACHE_DIR": "model_cache"}` in `pipeline_config` and load the model with  `pipe = ov_genai.LLMPipeline(model_path, "NPU", **pipeline_config)`
- NPU pipeline_config options:
  - `{"GENERATE_HINT": "BEST_PERF"}` or `{"GENERATE_HINT": "FAST_COMPILE"}` for optimizing for inference performance or model loading/compilation time.
  - `{"MAX_PROMPT_LEN": 2048, "MIN_RESPONSE_LEN": 512}`. `MAX_PROMPT_LEN` is 1024 by default, `MIN_RESPONSE_LEN` 128. If your input size is larger than 1024, or you expect more than 128 tokens in the output, set these options. `MIN_RESPONSE_LEN` will not cause more tokens to be generated than specified with `config.max_new_tokens`, and generating is still stopped if an EOS token is encountered.


Example `optimum-cli` command to export an NPU friendly model:

```sh
optimum-cli export openvino -m meta-llama/Llama-2-7b-chat-hf --weight-format int4 --sym --group-size -1 --ratio 1.0 --awq --scale-estimation --dataset wikitext2 Llama-2-7b-chat-hf-ov-int4
```

Use `optimum-cli export openvino --help` to see all options.

## Known issues

- With OpenVINO 2025.1 (and earlier), there is an issue when running inference on per-channel quantized INT4 models (exported with `optimum-cli export openvino --group-size -1`) on Meteor Lake GPU. The model generates nonsense. This issue is fixed in [nightly](#nightly)
- On GPU, first inference is expected to be different from subsequent inferences. If first inference is worse than second inference, please report an [issue](https://github.com/openvinotoolkit/openvino/issues). To prevent this variability, it can be useful to add a warmup inference: `pipe.generate("hello", max_new_tokens=1)`.
