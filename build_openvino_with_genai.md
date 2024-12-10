# Build OpenVINO with OpenVINO GenAI

## Windows

See https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/build_windows.md for full OpenVINO build instructions

### System requirements

- Microsoft Visual Studio 2019 or higher, version 16.3 or later
- CMake 3.23 or higher (Visual Studio 2022 version is fine)
- Git for Windows
- [NSIS](https://nsis.sourceforge.io)
- [WGET](https://eternallybored.org/misc/wget/)

### Build OpenVINO

- OpenVINO will be installed in `C:\Users\UserName\tools\openvino`. Modify the `--prefix` argument in the `--install` line to change that.
- We build a very minimal OpenVINO build, disabling among others AUTO, Python and NPU support. To enable Python, remove `-DENABLE_PYTHON=OFF`, to enable NPU support remove `-DENABLE_INTEL_NPU=OFF` etc. See [CMake options for custom compilation] for an overview of options. (https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/cmake_options_for_custom_compilation.md)
- To prevent potential environment conflicts, it is recommended to open a new Developer Command Prompt to run these steps.

> NOTE: This example assumes that you have not cloned OpenVINO and OpenVINO GenAI yet. If you already cloned them, run `git pull` and `git submodule update --init --recursive` for both repositories. The `cmake` command below assumes that `openvino` and `openvino.genai` are both cloned in the same parent directory. If that is not the case, adjust the path for `-DOPENVINO_EXTRA_MODULES` to the path to your `openvino.genai` repository.

> NOTE: if you're running into issues, you may want to set an absolute path to openvino.genai as argument to `-DOPENVINO_EXTRA_MODULES` in the `cmake` command below.

```shell
git clone --recursive https://github.com/openvinotoolkit/openvino.genai.git
git clone --recursive https://github.com/openvinotoolkit/openvino.git
cd openvino
mkdir build && cd build
cmake -G "Visual Studio 17 2022" -DENABLE_AUTO=OFF -DENABLE_AUTO_BATCH=OFF -DENABLE_PROXY=OFF -DENABLE_DOCS=OFF -DENABLE_OV_ONNX_FRONTEND=OFF -DENABLE_OV_PADDLE_FRONTEND=OFF -DENABLE_OV_TF_FRONTEND=OFF -DENABLE_OV_TF_LITE_FRONTEND=OFF -DENABLE_OV_PYTORCH_FRONTEND=OFF -DENABLE_OV_JAX_FRONTEND=OFF -DENABLE_MULTI=OFF -DENABLE_HETERO=OFF -DENABLE_TEMPLATE=OFF -DENABLE_PYTHON=OFF -DENABLE_WHEEL=OFF -DENABLE_SAMPLES=OFF -DENABLE_INTEL_NPU=OFF -DENABLE_FASTER_BUILD=ON -DOPENVINO_EXTRA_MODULES='..\..\openvino.genai' ..
cmake --build . --config Release --verbose --parallel
cmake --install . --prefix %USERPROFILE%\tools\openvino
```

### Use OpenVINO

Run `%USERPROFILE%\tools\openvino\setupvars.bat` (setupvars.ps1 for PowerShell) and compile your application

> [!NOTE]
> If you want to use your build with Python and use OpenVINO GenAI, set `-DENABLE_PYTHON=ON` when running cmake and run `pip install --no-deps openvino-tokenizers numpy`. OpenVINO Tokenizers needs to be the same version as OpenVINO, so if you're not building from a release tag, use `pip install --pre --no-deps --upgrade openvino-tokenizers --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly` and specify a version (i.e. `openvino-tokenizers==2025.0.*` if needed). Note the `no-deps` in the pip install commands! It is recommended to create a clean virtual environment to install these dependencies.


## Linux

See https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/build_linux.md for full OpenVINO build instructions.

### System requirements

- [CMake](https://cmake.org/download/) 3.23 or higher
- GCC 7.5 or higher
- Git

### Build OpenVINO

- OpenVINO will be installed in `$HOME/tools/openvino`. Modify the `--prefix` argument in the `--install` line to change that.
- We build a very minimal OpenVINO build, disabling among others AUTO, Python and NPU support. GPU support is enabled in the build, provided GPU drivers are installed. To enable Python, remove `-DENABLE_PYTHON=OFF`, to enable NPU support remove `-DENABLE_INTEL_NPU=OFF` etc. See [CMake options for custom compilation] for an overview of options. (https://github.com/openvinotoolkit/openvino/blob/master/docs/dev/cmake_options_for_custom_compilation.md)
- To prevent potential environment conflicts (for example from a Python virtual environment, or from previously ran setupvars), it is recommended to open a new terminal to run these steps.

> NOTE: This example assumes that you have not cloned OpenVINO and OpenVINO GenAI yet. If you already cloned them, run `git pull` and `git submodule update --init --recursive` for both repositories. The `cmake` command below assumes that `openvino` and `openvino.genai` are both cloned in the same parent directory. If that is not the case, adjust the path for `-DOPENVINO_EXTRA_MODULES` to the path to your `openvino.genai` repository.

```shell
git clone --recursive https://github.com/openvinotoolkit/openvino.genai.git
git clone --recursive https://github.com/openvinotoolkit/openvino.git
cd openvino
sudo ./install_build_dependencies.sh
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DENABLE_AUTO=OFF -DENABLE_AUTO_BATCH=OFF -DENABLE_PROXY=OFF -DENABLE_DOCS=OFF -DENABLE_OV_ONNX_FRONTEND=OFF -DENABLE_OV_PADDLE_FRONTEND=OFF -DENABLE_OV_TF_FRONTEND=OFF -DENABLE_OV_TF_LITE_FRONTEND=OFF -DENABLE_OV_PYTORCH_FRONTEND=OFF -DENABLE_OV_JAX_FRONTEND=OFF -DENABLE_MULTI=OFF -DENABLE_HETERO=OFF -DENABLE_TEMPLATE=OFF -DENABLE_PYTHON=OFF -DENABLE_WHEEL=OFF -DENABLE_SAMPLES=OFF -DENABLE_INTEL_NPU=OFF -DENABLE_FASTER_BUILD=ON -DCMAKE_SKIP_INSTALL_RPATH=OFF -DENABLE_LIBRARY_VERSIONING=OFF -DOPENVINO_EXTRA_MODULES=$(pwd)/../../openvino.genai ..
cmake --build . --parallel
cmake --install . --prefix $HOME/tools/openvino
```

### Use OpenVINO 

Run `source $HOME/tools/openvino/setupvars.sh` and compile your application

> [!NOTE]
> If you want to use your build with Python and use OpenVINO GenAI, set `-DENABLE_PYTHON=ON` when running cmake and run `pip install --no-deps openvino-tokenizers numpy`. OpenVINO Tokenizers needs to be the same version as OpenVINO, so if you're not building from a release tag, use `pip install --pre --no-deps --upgrade openvino-tokenizers --extra-index-url https://storage.openvinotoolkit.org/simple/wheels/nightly` and specify a version (i.e. `openvino-tokenizers==2025.0.*` if needed). Note the `no-deps` in the pip install commands! It is recommended to create a clean virtual environment to install these dependencies.
