# 3D Scene Reconstruction via NeRF & Gaussian Splatting

> An end-to-end pipeline exploration for novel view synthesis using Neural Radiance Fields and Gaussian Splatting with detailed failure analysis and MLOps learnings.

---

## Overview

This project attempts to reconstruct a 3D scene from a handheld smartphone video using **COLMAP** for camera pose estimation, followed by **NeRF** and **Gaussian Splatting** for novel view synthesis via **nerfstudio**. While the full pipeline was not completed due to COLMAP reconstruction failure, the project produced significant practical insights into real-world 3D reconstruction pipelines and MLOps debugging in headless environments.

> **Note:** This is an honest documentation of an incomplete pipeline. The value here is in the debugging journey, failure analysis, and practical MLOps learnings — not a polished result.

---

## What Was Accomplished

| Stage | Status | Notes |
|---|---|---|
| Video capture & preprocessing | Complete | 1:55 video, 6,921 frames @ 60fps, multi-scale extraction |
| Frame extraction (FFmpeg) | Complete | ~150–200 frames at 4 resolutions |
| COLMAP feature extraction | Complete | SIFT features extracted (CPU mode) |
| COLMAP feature matching | Complete | Exhaustive matching completed |
| COLMAP sparse reconstruction | Failed | "No good initial image pair found" |
| NeRF / Gaussian Splatting training | Blocked | Requires `transforms.json` from COLMAP |

---

## Root Cause Analysis

**Why COLMAP failed:**

1. **Insufficient viewpoint diversity** — handheld circular motion lacked the baseline and parallax needed to initialize structure-from-motion. The informal capture trajectory didn't provide enough geometric variation between frames.
2. **Repetitive textures** — book spines produced ambiguous SIFT features, confusing feature matching and preventing reliable pose estimation.
3. **Frame redundancy** — 60fps capture produced highly similar consecutive frames. Even after temporal downsampling to ~200 frames, viewpoint diversity remained too low.

**Infrastructure challenges encountered across 6 attempts:**

| Attempt | Environment | Issue | Resolution |
|---|---|---|---|
| 1–2 | Google Colab | COLMAP Qt GUI error in headless env | Manual COLMAP invocation with CPU mode |
| 3 | Google Colab | `ns-process-data` failed silently | Direct FFmpeg extraction bypass |
| 4 | Colab + manual COLMAP | Mapper: "No good initial image pair" | Varied frame counts, resolutions — same result |
| 5 | Windows local | Python 3.14.2 incompatible with nerfstudio; missing C++ build tools | Abandoned for cloud |
| 6 | Kaggle | COLMAP 3.13 / nerfstudio 0.3.4 version mismatch; OpenGL context failure | Patched version detection; no fix for headless GL |

---

## Key Learnings

1. **Capture quality matters as much as the model** — NeRF and Gaussian Splatting are only as good as the camera pose estimates. Planned circular trajectories with sufficient baseline are non-negotiable.
2. **Headless environments need explicit GUI bypass** — always audit Qt/OpenGL dependencies before running 3D reconstruction tools in cloud environments.
3. **Dependency chains are fragile** — nerfstudio + COLMAP + PyTorch + FFmpeg create a complex dependency graph. Docker containerization would have prevented most compatibility issues.
4. **Negative results are real results** — systematic failure analysis is a core engineering skill. Knowing *why* something doesn't work is often more valuable than a black-box success.

---

## Repository Structure

```
nerf-gaussian-splatting/
│
├── notebooks/
│   ├── 01_initial_nerfstudio_pipeline.ipynb       # First end-to-end attempt
│   ├── 02_manual_colmap_extraction.ipynb          # Manual COLMAP with custom frame extraction
│   ├── 03_ffmpeg_direct_extraction.ipynb          # Direct FFmpeg bypass approach
│   ├── 04_manual_colmap_reconstruction.ipynb      # Full manual COLMAP pipeline
│   ├── 05_colmap_reinstall_pipeline.ipynb         # Clean reinstall + retry
│   └── 06_final_debug_attempt.ipynb               # Final systematic debug
├── report/
│   └── NeRF_vs_Gaussian_Splatting_Report.docx     # Full project report
├── requirements.txt
└── README.md
```

---

## Setup (For Future Attempts)

```bash
git clone https://github.com/JananiV3010/nerf-gaussian-splatting.git
cd nerf-gaussian-splatting
pip install nerfstudio
# Requires COLMAP installed via system package manager
# Run in environment with display server or use --headless flag
```

**Recommended capture protocol for success:**
- Planned circular trajectory around object (not handheld freeform)
- 30–60 fps, minimum 200 frames with viewpoint diversity
- Sufficient lighting, non-repetitive textures
- Use `ns-process-data video` with `--num-frames-target 150`

---

## Tech Stack

`Python` `nerfstudio` `COLMAP` `FFmpeg` `PyTorch` `Google Colab` `Kaggle`

---

## Author

**Janani Vaiyapuriappan**, MSE Biomedical Engineering, Johns Hopkins University  
[LinkedIn](https://www.linkedin.com/in/janani-vaiyapuriappan/) · [GitHub](https://github.com/JananiV3010)

---

*Course project — Machine Perception, Johns Hopkins University (Fall 2025)*
