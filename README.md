
# LlamaKaggleRunner

**Deploy and run** any GGUF-based large language model (such as [Qwen](https://github.com/QwenLM/Qwen), Llama, Mistral, Gemma, and others) supported by **[llama.cpp](https://github.com/ggerganov/llama.cpp)** on dual Tesla T4 Kaggle/Colab instances with high-performance GPU offloading.

**Access securely** via persistent remote environments, integrated **[ttyd](https://github.com/tsl0922/ttyd)** web terminal access, and stable external proxying via **[pyngrok](https://github.com/alexdlaird/pyngrok)**.

---

## How to use

**Open the notebook in Kaggle or Google Colab and follow the step-by-step cells.**

1. Configure your desired model repository (`REPO_ID`), model filename (`MODEL_FILENAME`), and ngrok credentials directly in the configuration panel of the first cell.
2. Run the cells sequentially to provision the dual-GPU cloud environment, download/build `llama-server`, fetch the model from Hugging Face, and expose the API and web UI endpoints.

---

## Features

* **Universal GGUF Support**: Compatible with virtually any open-source model architecture supported by `llama.cpp` (e.g., Qwen, Llama 3, Mistral, Command R, Phi).
* **Dual-GPU Acceleration**: Split heavy model layers across two Tesla T4 GPUs (`--tensor-split 0.5,0.5`) to handle large-scale weights seamlessly.
* **Massive Context Support**: Run up to **128k+ tokens** (`--ctx-size 131072`) with automated key-value cache quantization (`--cache-type-k q4_0 --cache-type-v q4_0`) to optimize VRAM consumption.
* **Session Persistence**: Automatically save and restore model chat states and sessions directly to disk (`--session /kaggle/working/chat_session.bin`).
* **Remote API & WebUI**: Built-in OpenAI-compatible `/v1` endpoint for seamless integration with IDE extensions (Cursor, VS Code / Continue, Aider) alongside a native browser WebUI.
* **Browser Terminal (`ttyd`)**: Integrated web terminal access on port 9000 for managing files and server execution on the fly.

---

## Configuration & Model Customization

You can easily swap out models by modifying the configuration variables at the top of the first notebook cell:

```python
# Configuration panel example
REPO_ID = "bartowski/Qwen_Qwen3.6-35B-A3B-GGUF"  # Or any other Hugging Face GGUF repository
MODEL_FILENAME = "Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf" # Desired quantization file

```

---

## Local Execution & Manual Usage

If you prefer to run or adapt the build workflow locally or inside a custom container environment:

1. Clone this repository or download the required script components.
2. Ensure you have `cmake`, `build-essential`, `ccache`, `pigz`, and CUDA toolkit dependencies installed.
3. Configure your API tokens and environment variables in the setup block.

### Running `llama-server` manually

```sh
# Run llama-server with dual-GPU tensor split, 128k context, and 4-bit KV cache quantization
./llama.cpp/build/bin/llama-server \
  --model /tmp/models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
  --ctx-size 131072 \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --n-gpu-layers 99 \
  --tensor-split 0.5,0.5 \
  --port 8000 \
  --host 0.0.0.0 \
  --session /kaggle/working/chat_session.bin

```

### CLI Options Reference

```
  --model MODEL                 path to GGUF model file
  --ctx-size N                  size of the prompt context (default: 512, supports up to 131072+)
  --cache-type-k TYPE           KV cache key type (f16, q8_0, q4_0)
  --cache-type-v TYPE           KV cache value type (f16, q8_0, q4_0)
  --n-gpu-layers N              number of layers to store in VRAM
  --tensor-split SPLIT          fraction of the model to offload to each GPU (e.g., 0.5,0.5)
  --port PORT                   port to listen on (default: 8080 / 8000)
  --host HOST                   ip address to listen on (default: 0.0.0.0)
  --session FNAME               file to cache/load session state for persistence

```

---

## Terminal Access via `ttyd`

The environment includes an automated startup script for `ttyd` running on port 9000. When connected via `pyngrok`, it provides a fully interactive bash shell directly in your web browser, allowing you to monitor resource usage (`nvidia-smi`), inspect disk logs, and manage session storage inside `/kaggle/working`.
