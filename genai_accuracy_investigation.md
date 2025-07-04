# OpenVINO LLM accuracy investigation suggestions

This document provides some tips for investigating accuracy issues with LLMs with OpenVINO on AI PC. Some of these options may improve accuracy, others may help
pinpoint where the issue is. I am not necessarily recommending all these steps for all issues; it is a list of suggestions to consider.

## Check on CPU

If the issue occurs on GPU/NPU, it is useful to first check model outputs on CPU.

Flowchart:
- Outputs good on CPU, bad on GPU and/or NPU -> try Drivers, OpenVINO GenAI version, Model export versions and OpenVINO model compilation properties section
- Outputs good on CPU and GPU, bad on NPU -> try Drivers, OpenVINO GenAI and Model export versions sections. If the model is supported on NPU and latest drivers/versions do not work, report an issue.
- Outputs bad everywhere -> first try on CPU with `EXECUTION_MODE_HINT='ACCURACY'`. If issue is not fixed, try all other sections.

## Drivers

For GPU and NPU, make sure that the latest drivers are installed.

## OpenVINO GenAI version

Try with the latest OpenVINO GenAI release, and with the latest nightly version:

```
pip install --pre -U openvino-genai --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly          
```

## OpenVINO model compilation properties

By default, OpenVINO enables settings that optimize performance and do not impact accuracy significantly. These settings may not be ideal for every model, 
and when trying to pinpoint issues with model outputs, particularly with quantized models, it can be useful to modify these settings. They can be
passed as parameters to OpenVINO GenAI's `LLMPipeline()`. 

To see the settings that are currently enabled for a model on a particular device, see [show_compiled_model_properties.py](https://github.com/helena-intel/snippets/blob/main/show_properties/show_compiled_model_properties.py)

Example, assuming your local model is `phi-4-mini-instruct-ov-int4`:
```
curl -s https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/show_properties/show_compiled_model_properties.py | python - phi-4-mini-instruct-ov-int4 GPU
```

The following settings can impact accuracy. They can be changed for CPU and GPU.
- `EXECUTION_MODE_HINT`. This is a high level setting which on GPU and Xeon CPU sets INFERENCE_PRECISION_HINT to f32 and disables dynamic quantization.
   This is a recommended first method to try, instead of the individual options below, but it can fail if there is an issue with f32 inference on a device where that is not the default.
- `INFERENCE_PRECISION_HINT` (GPU, Xeon CPU). This is set to `f16`/`bf16` on GPU/Xeon CPU. Setting it to `f32` may improve accuracy, at the cost of slower latency (particularly for first inference) and higher memory use. Some models may not work with f32 inference precision on GPU.
- `DYNAMIC_QUANTIZATION_GROUP_SIZE`. Dynamic quantization is enabled by default on some devices. Setting `DYNAMIC_QUANTIZATION_GROUP_SIZE` to `0` disables dynamic quantization.
- `KV_CACHE_PRECISION`. If this is set to int8, setting it to `f16` may improve accuracy. On CPU, an advanced option is to set `KEY_CACHE_PRECISION`, `KEY_CACHE_GROUP_SIZE`, `VALUE_CACHE_PRECISION` and `VALUE_CACHE_GROUP_SIZE` individually.

## First inference

For GPU and NPU, it is expected that first inference can be different. It should not be worse than second inference (this is considered a bug),
but his can happen, so it is useful to check if the accuracy issue occurs only on first inference, or also on subsequent inferences. If the issue is with first inference only,
a workaround is to do a warmup inference: `pipe.generate("hello", max_new_tokens=1)`.

## Model export

### Model export versions

If a model has been exported a while ago, it is possible that there was a known issue that has since been fixed. [modelinfo.py](https://github.com/helena-intel/snippets/blob/main/model_info/modelinfo.py
) shows the dependencies that were in used when exporting the model.

For example:
```
curl -s https://raw.githubusercontent.com/helena-intel/snippets/refs/heads/main/model_info/modelinfo.py | python - phi-4-mini-instruct-ov-int4
```

If the model has been exported with older versions of optimum-intel/OpenVINO/NNCF, it can be worth trying to install optimum-intel from source with all the latest dependencies, and export the model again

```
python -m pip install --upgrade --upgrade-strategy eager "optimum-intel[openvino]"@git+https://github.com/huggingface/optimum-intel.git
```

### Model export parameters

For GPU and CPU:

- If the model is small and the inference device is relatively recent (for example a 3B model on LNL GPU), INT8 models may work fine. INT8 will generally have better accuracy than INT4. Optimum Intel exports models to INT8 by
  default, so if you export a model with no parameters, you will get a model with INT8 compressed weights. You can also specify `--weight-format int8`.

- For many popular models, default optimized quantization parameters are available and it is worth it to first export the model with these settings. To do that, use
  ```
  optimum-cli export openvino --weight-format int4
  ```
  without specifying any other parameters (no `--sym`, no `--ratio` etc.).

  If that does not help, in general, asymmetric quantization with lower group sizes preserves accuracy better than symmetric quantization with channel wise quantization or higher group sizes. It is also possible to specify a ratio to keep more
  parameters in INT8. For inference on CPU or GPU, it can therefore be worth trying to export the model with

  ```
  optimum-cli export openvino --weight-format int4 --group-size 32 --ratio 0.8 --awq --dataset wikitext2 --scale-estimation
  ```

## Generation parameters and code

- Some models are very sensitive to generation parameters. When investigating accuracy, it can be useful to try with greedy search, with only `do_sample=False` and no other parameters. If quality is better with this config,
experiment with changing one setting at a time.
- Some issues happen in "chat mode" but not outside of it. When running into issues, it can be useful to run inference both with `pipe.start_chat()`/`pipe.finish_chat()` (keeping chat history) and without it.

## System prompt

If the output is not terrible, but not good either, modifying the system prompt to be explicit about what you expect the model to do can be very effective. Use `pipe.start_chat(system_message="your system prompt")` 
or set a system prompt with [manual chat tokenization](https://github.com/helena-intel/snippets/blob/main/llm_chat/python/llm_chat_manual.py).
