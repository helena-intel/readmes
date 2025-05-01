# Environment variables

`OPENVINO_LOG_LEVEL=3` with OpenVINO GenAI, set to 3 to show compiled model and device properties when loading models
`ONEDNN_MAX_CPU_ISA=AVX512_CORE_VNNI` disable BF16 on Xeon 4 and higher. See https://www.intel.com/content/www/us/en/docs/onednn/developer-guide-reference/2023-0/cpu-dispatcher-control.html
`DNNL_VERBOSE=1` show verbose OneDNN log, for example to confirm that BF16 is being used on Xeon 4 and higher.
