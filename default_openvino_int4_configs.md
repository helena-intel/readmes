# Default OpenVINO quantization configs for INT4 quantization

This page lists the default quantization configurations for OpenVINO LLMs. These configs are used when using `optimum-cli` with
`--weight-format int4` without specifying additional quantization parameters:

```
optimum-cli export openvino -m model_id --weight-format int4 model-id-ov-int4
```

The table shows settings that are explicitly set. If a cell is empty, the setting is not set. For example, for the
`HuggingFaceH4/zephyr-7b-beta` model, `AWQ` is set, but `scale_estimation` is not, and no dataset is specified for AWQ (data-free AWQ will
be used). The default config for this model is equivalent to doing

```
optimum-cli export openvino -m HuggingFaceH4/zephyr-7b-beta --sym --group-size 128 --ratio 0.8 --awq zephyr-7b-beta-ov-int4
```

| model                                         | sym   |   group_size | ratio   | quant_method   | dataset   | scale_estimation   | all_layers   |
|:----------------------------------------------|:------|-------------:|:--------|:---------------|:----------|:-------------------|:-------------|
| baichuan-inc/Baichuan2-13B-Chat               | False |          128 | 1.0     | AWQ            |           |                    |              |
| baichuan-inc/Baichuan2-7B-Chat                | False |          128 | 0.8     |                |           |                    |              |
| bigcode/starcoder2-15b                        | False |           64 | 1.0     |                |           |                    |              |
| bigcode/starcoder2-3b                         | False |          128 | 0.9     |                |           |                    |              |
| bigscience/bloomz-560m                        | True  |           64 | 0.8     | AWQ            | wikitext2 |                    |              |
| databricks/dolly-v2-3b                        | False |          128 | 1.0     |                | wikitext2 | True               |              |
| deepseek-ai/DeepSeek-R1-Distill-Llama-8B      | False |           64 | 0.8     | AWQ            |           |                    |              |
| deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B     | False |           32 | 0.7     | AWQ            |           |                    |              |
| deepseek-ai/DeepSeek-R1-Distill-Qwen-7B       | False |          128 | 1.0     | AWQ            |           |                    |              |
| EleutherAI/gpt-j-6b                           | False |           64 |         |                |           |                    |              |
| facebook/opt-2.7b                             | True  |          128 | 0.7     |                |           |                    |              |
| facebook/opt-6.7b                             | False |           64 | 0.8     |                |           |                    |              |
| HuggingFaceH4/zephyr-7b-beta                  | True  |          128 | 0.8     | AWQ            |           |                    |              |
| lmsys/longchat-7b-16k                         | False |          128 | 1.0     | AWQ            | wikitext2 | True               |              |
| lmsys/vicuna-7b-v1.5                          | False |          128 | 1.0     |                |           |                    |              |
| meta-llama/Llama-2-13b-chat-hf                | True  |           64 | 0.8     |                |           |                    |              |
| meta-llama/Llama-2-7b-chat-hf                 | True  |          128 | 1.0     | AWQ            |           |                    |              |
| meta-llama/Llama-2-7b-hf                      | True  |          128 | 0.6     |                |           |                    |              |
| meta-llama/Llama-3.1-8B                       | False |           64 | 0.8     | AWQ            |           |                    |              |
| meta-llama/Llama-3.1-8B-Instruct              | False |           64 | 0.8     | AWQ            |           |                    |              |
| meta-llama/Llama-3.2-1B-Instruct              | False |          128 | 1.0     | AWQ            |           |                    |              |
| meta-llama/Meta-Llama-3.1-8B                  | False |           64 | 0.8     | AWQ            |           |                    |              |
| meta-llama/Meta-Llama-3.1-8B-Instruct         | False |           64 | 0.8     | AWQ            |           |                    |              |
| microsoft/phi-2                               | False |           64 | 1.0     | AWQ            | wikitext2 | True               |              |
| microsoft/Phi-3-mini-4k-instruct              | False |           64 | 1.0     |                | wikitext2 | True               |              |
| microsoft/Phi-3.5-mini-instruct               | False |           64 | 1.0     | AWQ            |           |                    |              |
| microsoft/Phi-4-mini-instruct                 | False |           64 | 1.0     | AWQ            |           |                    |              |
| mistralai/Mistral-7B-v0.1                     | True  |          128 | 0.9     |                |           |                    |              |
| mistralai/Mixtral-8x7B-v0.1                   | True  |          128 | 0.8     |                |           |                    |              |
| openlm-research/open_llama_3b                 | False |           64 |         |                |           |                    | True         |
| openlm-research/open_llama_3b_v2              | False |           64 | 1.0     | AWQ            | wikitext2 |                    |              |
| pansophic/rocket-3B                           | True  |          128 | 0.8     |                |           |                    |              |
| psmathur/orca_mini_3b                         | True  |           64 |         |                |           |                    | True         |
| Qwen/Qwen-7B-Chat                             | True  |          128 | 0.6     |                |           |                    |              |
| Qwen/Qwen2.5-1.5B-Instruct                    | False |          128 | 0.9     | AWQ            | wikitext2 | True               |              |
| Qwen/Qwen2.5-7B-Instruct                      | False |          128 | 1.0     | AWQ            |           |                    |              |
| Qwen/Qwen2.5-Coder-3B-Instruct                | False |           64 | 1.0     |                | wikitext2 | True               |              |
| Qwen/Qwen3-1.7B                               | True  |           64 | 1.0     | AWQ            | wikitext2 | True               |              |
| Qwen/Qwen3-4B                                 | True  |          128 | 1.0     | AWQ            |           |                    |              |
| Qwen/Qwen3-8B                                 | False |          128 | 1.0     |                | wikitext2 | True               |              |
| stabilityai/stable-code-3b                    | True  |           64 | 0.8     |                |           |                    |              |
| stabilityai/stablelm-3b-4e1t                  | True  |           64 | 0.8     | AWQ            |           |                    |              |
| stabilityai/stablelm-tuned-alpha-3b           | False |          128 | 0.8     |                |           |                    |              |
| stabilityai/stablelm-tuned-alpha-7b           | False |           64 | 1.0     |                | wikitext2 | True               |              |
| stabilityai/stablelm-zephyr-3b                | False |          128 | 0.9     |                | wikitext2 | True               |              |
| THUDM/chatglm2-6b                             | True  |          128 | 0.72    |                |           |                    |              |
| tiiuae/falcon-7b-instruct                     | False |           64 |         |                |           |                    |              |
| TinyLlama/TinyLlama-1.1B-Chat-v1.0            | False |           64 | 1.0     | AWQ            |           |                    |              |
| togethercomputer/RedPajama-INCITE-7B-Instruct | False |          128 |         |                |           |                    |              |
| togethercomputer/RedPajama-INCITE-Chat-3B-v1  | False |          128 | 1.0     |                | wikitext2 | True               |              |

[Source](https://github.com/huggingface/optimum-intel/blob/main/optimum/intel/openvino/configuration.py#L54)

[`optimum-cli export openvino` documentation](https://huggingface.co/docs/optimum/main/en/intel/openvino/export)
