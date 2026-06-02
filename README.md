# Foley-Omni

[![arXiv](https://img.shields.io/badge/arXiv-Paper-brightgreen.svg?style=flat-square)](ARXIV_LINK)
[![Project Page](https://img.shields.io/badge/Project%20Page-Demo-purple?style=flat-square)](PROJECT_PAGE_LINK)
[![Hugging Face Models](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Models-blue?style=flat-square)](https://huggingface.co/CocoBro/Foley-Omni)
[![Demo](https://img.shields.io/badge/Demo-Video-orange?style=flat-square)](https://github.com/ty0402/Foley-Omni/issues/1)

## Demo

<p align="center">
  <video
    src="https://github.com/user-attachments/assets/14ed8124-04d5-4333-89f9-4fd699e93d98"
    controls
    autoplay
    muted
    loop
    playsinline
    width="85%">
  </video>
</p>

## Introduction

**Foley-Omni** is a unified multimodal generation model for **Video-to-Soundtrack (V2ST)**.  
Given a video and text conditions, Foley-Omni jointly generates synchronized **speech**, **sound effects**, and **music**. Besides complete soundtrack generation, the public release also supports task-level generation for **speech synthesis**, **sound effect generation**, and **music composition**.

---

## Model Download

| Models | 🤗 Hugging Face |
|-------|-------|
| Foley-Omni | [CocoBro/Foley-Omni](https://huggingface.co/CocoBro/Foley-Omni) |

Download the released checkpoints into `./ckpts/` with:

```bash
bash scripts/download_release_ckpts.sh CocoBro/Foley-Omni
```

Expected checkpoint layout:

```text
ckpts/
├── Foley-Omni/
│   └── v2st.pth
├── Wan2.2-TI2V-5B/
│   ├── models_t5_umt5-xxl-enc-bf16.pth
│   └── google/
│       └── umt5-xxl/
│           ├── special_tokens_map.json
│           ├── spiece.model
│           ├── tokenizer.json
│           └── tokenizer_config.json
└── mmaudio/
    └── ext_weights/
        ├── v1-16.pth
        ├── best_netG.pt
        └── synchformer_state_dict.pth
```

---

## Model Usage

### Dependencies and Installation

- Python >= 3.10
- [PyTorch 2.6.0](https://pytorch.org/)
- CUDA 12.4
- FlashAttention 2.7.4.post1

```bash
# 1. Clone the repository
git clone https://github.com/ty0402/Foley-Omni.git
cd Foley-Omni

# 2. Create environment
conda create -n foley-omni python=3.10 -y
conda activate foley-omni

# 3. Install PyTorch and dependencies
pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cu124
pip install -r requirements.txt
pip install flash_attn==2.7.4.post1 --no-build-isolation
pip install -U "huggingface_hub[cli]"

# 4. Download released checkpoints
bash scripts/download_release_ckpts.sh CocoBro/Foley-Omni
```

### Quick Start

#### 1. Video-to-Soundtrack Inference

The public checkpoint is designed for videos **up to 10 seconds**.

```bash
python inference_v2st.py --config-file inference_v2st.yaml
```

The default batch example is:

- [examples/video_text_example.json](./examples/video_text_example.json)

Generated files are written to `./outputs/v2st/`, including:

- `*.mp4`: input video merged with the generated soundtrack
- `pred_mapping.jsonl`: output mapping file
- `inference_config.yaml`: saved runtime config

#### 2. Single-Video Inference

Edit [inference_v2st.yaml](./inference_v2st.yaml):

1. Disable `json_file`
2. Set `video_path`
3. Set `text_prompt`

Then run:

```bash
python inference_v2st.py --config-file inference_v2st.yaml
```

#### 3. Text-Only Generation

Representative text-only prompts are provided at:

- [examples/text_example.jsonl](./examples/text_example.jsonl)

Run:

```bash
python inference.py --config-file inference_fusion.yaml
```

Outputs are written to `./outputs/text_only/` as `*.wav` files.

#### 4. Setup Check

Before inference, you can verify that the required local checkpoints are in place:

```bash
python scripts/check_setup.py
```

---

## Input Format

The batch V2ST input file is a JSON dictionary:

```json
{
  "./examples/videos/721ecf7c92d162bd2d74820f72f68d41.mp4": {
    "resp": "[WORDS]That car came by faster than I expected.[END_WORDS][AUDIO_CAPTION]A clear, neutral English-speaking voice is accompanied by the sound of a car passing on a quiet urban street.[END_AUDIO_CAPTION]"
  }
}
```

Each key is a video path. Each value is a metadata object for soundtrack generation.

Supported fields:

- `resp`: required structured prompt string
- `clip_feature_path`: optional pre-extracted CLIP feature path
- `sync_feature_path`: optional pre-extracted Sync feature path

The `resp` field can contain any subset of the following blocks:

- `[WORDS] ... [END_WORDS]`: speech content
- `[AUDIO_CAPTION] ... [END_AUDIO_CAPTION]`: sound effects, actions, and acoustic events
- `[MUSIC] ... [END_MUSIC]`: background music style, mood, instrumentation, and tempo

Notes:

- At least one of `WORDS`, `AUDIO_CAPTION`, or `MUSIC` should be present.
- `clip_feature_path` and `sync_feature_path` are optional.
- If features are not provided, Foley-Omni extracts visual features online from the input video.

---

## Prepare Visual Features

To pre-extract CLIP and Sync features, use:

- [data_process/convert_memmap_to_npy.py](./data_process/convert_memmap_to_npy.py)

Example:

```bash
python data_process/convert_memmap_to_npy.py \
  --json_input ./examples/video_text_example.json \
  --feature_dir ./examples/features \
  --json_output ./examples/video_text_with_features.json \
  --gpu_ids 0
```

---

## Todo

- [x] Release model weights
- [x] Release inference code
- [ ] Release V2ST-Bench

---

## Acknowledgement

We thank the following open-source projects for their inspiration and code:

- [MMAudio](https://github.com/hkchengrex/MMAudio)
- [Ovi](https://github.com/Wan-Video/Ovi)
- [Wan2.2](https://github.com/Wan-Video/Wan2.2)

