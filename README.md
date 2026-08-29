# qwen_edit

**Qwen-Image-Edit 2509 in ComfyUI on Google Colab** — instruction-driven image
editing (change backgrounds, swap outfits, restyle scenes, add/remove objects,
edit text) served through a public URL, with models cached in Google Drive so
they download **once**.

This is the image-editing counterpart to the
[Diffusion](https://github.com/mmorrisj/Diffusion) repo (Wan 2.2 image-to-video):
same one-notebook, tunnel-to-a-public-URL pattern, retargeted at Qwen-Image-Edit.

## Which notebook

Two notebooks, same workflow and same UI — they differ only in where the weights live.

| Notebook | Base weights (~28 GB) | LoRAs | Drive used |
|---|---|---|---|
| [`qwen_image_edit_comfyui_colab.ipynb`](qwen_image_edit_comfyui_colab.ipynb) | Google Drive, downloaded once | Drive | ~29 GB |
| [`qwen_image_edit_comfyui_colab_hf.ipynb`](qwen_image_edit_comfyui_colab_hf.ipynb) | Hugging Face → runtime local disk, re-pulled each session | Drive | **LoRAs + images only** |

The `_hf` variant trades ~3–7 min per session (`hf_transfer` pulls all 28 GB at
100–300 MB/s) for ~28 GB of Drive quota. It is often no slower in practice, since
reading 28 GB back out of the Drive FUSE mount is not fast either and can trip
Drive's bandwidth quota. It also ships a *Reclaim Drive space* cell that deletes
base weights left behind by the original notebook — never touching `loras/`.

Either way your LoRAs, edits and input images stay in Drive.

## Quick start

1. Open one of the notebooks above in Google Colab.
2. `Runtime → Change runtime type → A100 GPU` (L4 also works; add `--lowvram` for smaller GPUs).
3. Run the cells top to bottom:
   - **Step 1** verify GPU
   - **Step 2** mount Google Drive (`MyDrive/ComfyUI_Qwen`)
   - **Step 3** install ComfyUI (+ ComfyUI-Manager)
   - **Step 4** symlink Drive model/output/input folders into ComfyUI
   - **Step 5** fetch the base models — cached in Drive after the first run, or
     pulled from Hugging Face each session in the `_hf` notebook
   - **Step 5b** LoRAs *(optional in the original; in `_hf` it also fetches the
     Lightning LoRA into Drive)*
   - **Step 6** install the bundled workflow
   - **Step 7** launch ComfyUI + public URL
   - **Step 8** open the workflow and run your edit
4. Edited images land in Drive `ComfyUI_Qwen/output/`.

## Models

All downloaded automatically, and skipped if already present. The original
notebook puts all four in Drive; the `_hf` notebook splits them — the first
three come from Hugging Face in Step 5, the LoRA is kept in Drive by Step 5b.

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
- Models persist in Drive, so subsequent sessions skip the big download. In the
  `_hf` notebook only the LoRAs persist; the base weights are re-pulled from
  Hugging Face each session and never written to Drive.
