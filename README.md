# qwen_edit

**Qwen-Image-Edit 2509 in ComfyUI on Google Colab** — instruction-driven image
editing (change backgrounds, swap outfits, restyle scenes, add/remove objects,
edit text) served through a public URL, with models cached in Google Drive so
they download **once**.

This is the image-editing counterpart to the
[Diffusion](https://github.com/mmorrisj/Diffusion) repo (Wan 2.2 image-to-video):
same one-notebook, Drive-persisted, tunnel-to-a-public-URL pattern, retargeted at
Qwen-Image-Edit.

## Quick start

1. Open **[`qwen_image_edit_comfyui_colab.ipynb`](qwen_image_edit_comfyui_colab.ipynb)** in Google Colab.
2. `Runtime → Change runtime type → A100 GPU` (L4 also works; add `--lowvram` for smaller GPUs).
3. Run the cells top to bottom:
   - **Step 1** verify GPU
   - **Step 2** mount Google Drive (`MyDrive/ComfyUI_Qwen`)
   - **Step 3** install ComfyUI (+ ComfyUI-Manager)
   - **Step 4** symlink Drive model/output/input folders into ComfyUI
   - **Step 5** download the Qwen-Image-Edit models (~30 GB, first run only)
   - **Step 5b** *(optional)* add your own LoRAs
   - **Step 6** install the bundled workflow
   - **Step 7** launch ComfyUI + public URL
   - **Step 8** open the workflow and run your edit
4. Edited images land in Drive `ComfyUI_Qwen/output/`.

## Models

All downloaded automatically in Step 5 (skipped if already in Drive):

| Component | File | ComfyUI folder | Size |
|---|---|---|---|
| Diffusion (edit) | `qwen_image_edit_2509_fp8_e4m3fn.safetensors` | `diffusion_models` | ~20 GB |
| Text encoder | `qwen_2.5_vl_7b_fp8_scaled.safetensors` | `text_encoders` | ~9 GB |
| VAE | `qwen_image_vae.safetensors` | `vae` | ~250 MB |
| Lightning 4-step LoRA | `Qwen-Image-Edit-2509-Lightning-4steps-V1.0-bf16.safetensors` | `loras` | ~850 MB |

Sources: [Comfy-Org/Qwen-Image-Edit_ComfyUI](https://huggingface.co/Comfy-Org/Qwen-Image-Edit_ComfyUI),
[Comfy-Org/Qwen-Image_ComfyUI](https://huggingface.co/Comfy-Org/Qwen-Image_ComfyUI),
[lightx2v/Qwen-Image-Lightning](https://huggingface.co/lightx2v/Qwen-Image-Lightning).

## Workflow

[`workflows/qwen_image_edit_2509_subgraph.json`](workflows/qwen_image_edit_2509_subgraph.json)
is the official Qwen-Image-Edit-2509 subgraph workflow, wired for the 4-step
Lightning LoRA (steps **4**, CFG **1.0**, `euler` / `simple`). It uses **only
ComfyUI core nodes** — no third-party custom nodes required — but needs a recent
ComfyUI for `TextEncodeQwenImageEditPlus` / `CFGNorm` (Step 3 pulls latest).

Step 6 copies it into ComfyUI's user workflows, so it appears under the
**Workflows** (📂) sidebar in the UI. Qwen-Image-Edit 2509 accepts **up to 3
input images** (main image + optional style/background references).

## Adding LoRAs

Either drop `.safetensors` files into Drive `ComfyUI_Qwen/models/loras/`, or add
`(url, filename)` entries to `EXTRA_LORAS` in **Step 5b**. Restart ComfyUI
(re-run Step 7) and point a `LoraLoaderModelOnly` node at the new file. Stack
multiple `LoraLoaderModelOnly` nodes to chain LoRAs. The official edit-LoRA pack
(Relight, Fusion, Multiple-angles, White-to-Scene, …) lives in
[Comfy-Org/Qwen-Image-Edit_ComfyUI `split_files/loras/`](https://huggingface.co/Comfy-Org/Qwen-Image-Edit_ComfyUI/tree/main/split_files/loras).

## Notes

- **VRAM:** the fp8 edit model wants ≥16 GB free. A100 (40 GB) is comfortable,
  L4 (24 GB) works, T4 (16 GB) is borderline — add `--lowvram` in Step 7.
- **Tunnels:** `colab` (default, no auth), `cloudflare`, or `ngrok`. See Step 7.
- Models persist in Drive, so subsequent sessions skip the big download.
