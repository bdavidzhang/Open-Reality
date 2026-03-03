<div align="center">

# Open Reality

### One phone. Any space. Real spatial intelligence.

**HackIllinois 2026 — 🥇 Modal Track · Best Use of Gemini API · Most Creative**

<br />

<img src="assets/vggt_slam_demo.gif" alt="Open Reality" width="95%"/>

<br />

Point any phone at a room — no app, no hardware — and a live 3D map builds in real time on Modal's H100 GPU. An AI agent finds objects by name, pins them in 3D, and answers spatial questions using real geometry.

<br />

[Live Demo](#live-demo) · [How It Works](#how-it-works) · [Deploy It Yourself](#deploy-it-yourself) · [Architecture](#architecture)

</div>

---

## The Problem

Every year, first responders walk into buildings they've never seen. Search-and-rescue teams navigate rubble without a map. Visually impaired people enter new spaces with no spatial context.

AI could help — if it could actually see.

Today's AI agents are spatially blind. They process pixels and text, but have no concept of *where* things are in 3D space. The tools that do provide spatial maps — LiDAR rigs, depth cameras, survey hardware — cost tens of thousands of dollars and require trained operators. Spatial intelligence has stayed locked inside robotics labs.

**Open Reality breaks that barrier.**

---

## What We Built

Open Reality is a cloud-native spatial AI platform. Describe your task. Point a phone — or drop a video, or upload a folder of images. An AI agent maps your space in real time, finds your targets, and answers spatial questions — from any browser, from anywhere.

### Input Modes

| Mode | How |
|------|-----|
| **Live Camera** | Open a link on any phone browser. No app install. Stream directly. |
| **Video File** | Upload a recorded walkthrough — MP4, MOV, or any standard format. |
| **Image Folder** | Drop a folder of frames (e.g. extracted with `ffmpeg`). Full offline SLAM. |

### The Experience

1. **Describe your mission** — *"I'm a paramedic doing a safety sweep."*
2. **Get a spatial plan** — the AI agent generates what to look for and how to move through the space.
3. **Capture the space** — live camera, uploaded video, or image folder.
4. **Walk the space** — a dense 3D point cloud builds live as you move.
5. **The agent finds your targets** — objects are detected, pinned in 3D, and available for spatial Q&A.

### Agentic Intelligence

Open Reality doesn't just map. It reasons about what it sees.

- **Intent-driven planning** — before a frame is captured, the agent interprets your goal and builds a typed spatial action plan. A firefighter's plan surfaces extinguishers and standpipe connections. A crime scene investigator's plan surfaces evidence markers. The agent understands the difference.
- **Continuous detection** — as the map grows, the agent automatically scans every new submap for your targets. No user action needed.
- **Retroactive re-search** — add a new target mid-scan and the agent immediately re-runs detection on everything it's already seen. Its knowledge of the space updates instantly, backwards in time.
- **Spatial Q&A** — after the scan, the AI holds the full 3D context. *"Is the fire extinguisher accessible from the north stairwell?"* gets answered with actual geometry — not a guess.

---

## Why Modal

The inference pipeline — a 1-billion-parameter vision model, CLIP scoring on every submap, SAM3 segmentation — is far too heavy for a laptop and too latency-sensitive for a slow API call. Modal makes real-time spatial AI possible.

| | |
|---|---|
| **H100 GPU** | Three heavy models in sequence at real-time speed. No dropped frames. |
| **Warm Containers** | No cold starts between WebSocket frames. Inference hits in under 100ms per submap. |
| **Modal Volumes** | VGGT-1B (4 GB) + DINO-Salad weights cached across runs. First inference in seconds, not minutes. |
| **Modal Tunnel** | Provides the HTTPS endpoint the phone camera requires to stream. Zero SSL config. |
| **One Command** | `modal deploy modal_streaming.py` — stable public URL, live for anyone. |

This is Modal the way it's meant to be used: serious inference, at real-time speed, accessible from a link.

---

## Demo

Deploy your own instance of OpenReality in one command and get a stable public URL instantly.

```bash
modal deploy modal_streaming.py
```

Modal will output a URL like `https://yourname--vggt-slam-streaming-web.modal.run`. Open that on any device. Point your phone at a room. Watch the 3D map build in real time. Ask the agent to find something.

No app. No hardware. Just a browser and your Modal account.

---

## Use Cases

The same platform, tuned to radically different missions — just by changing the prompt.

| Scenario | Mission Prompt | What the Agent Finds |
|----------|---------------|----------------------|
| **Active Fire Scene** | *"I'm a firefighter doing a sweep of a multi-story office building before full evacuation."* | Fire extinguishers, standpipe connections, exit signs, gas shutoff valves, victims |
| **Post-Natural Disaster** | *"I'm a search-and-rescue coordinator mapping a building after an earthquake."* | Structural damage, blocked exits, trapped victims, hazardous material spills, load-bearing failures |
| **Pre-Listing Property Walkthrough** | *"I'm a realtor doing a pre-listing inspection of a vacant house."* | Scuffed walls, dated fixtures, worn carpet, stained grout, foggy window panes |
| **Hotel Room Safety Inspection** | *"I'm a health inspector auditing a hotel room."* | Mold, missing smoke detector components, broken seals, window latch failures |
| **Post-Burglary Home Walkthrough** | *"I'm documenting damage after a break-in for an insurance claim."* | Forced entry points, ransacked areas, broken displays, emptied safes |
| **Construction Site Safety Audit** | *"I'm a site safety officer doing a compliance walkthrough."* | Exposed rebar, unsecured ladders, open floor penetrations, missing guardrails, blocked exits |
| **Flood-Damaged Home Inspection** | *"I'm a loss adjuster inspecting a home after indoor flooding."* | Waterlogged surfaces, swollen drywall, delaminating cabinets, soft subfloor, standing water |

The hardware barrier is gone. The deployment barrier is gone. What remains is the mission.

---

## Deploy It Yourself

### Prerequisites

- Python 3.11+
- [Modal](https://modal.com) account with GPU access
- [Conda](https://docs.conda.io/en/latest/) (recommended)

### Installation

```bash
git clone https://github.com/bdavidzhang/Real-Eyes.git
cd Real-Eyes

conda create -n open-reality python=3.11
conda activate open-reality

chmod +x setup.sh && ./setup.sh
```

The setup script installs all dependencies and clones third-party models (VGGT, DINO-Salad, Perception Encoder, SAM3) into `third_party/`.

### Cloud Deployment (Modal) — Recommended

```bash
# One-command production deployment — stable URL, always-on
modal deploy modal_streaming.py

# Development mode with auto-reload
modal serve modal_streaming.py

# Pre-cache model weights (optional, speeds up first run)
modal run modal_streaming.py::app.download_models
```

The streaming server keeps an H100 container warm, serves the frontend as static files built at image creation time, and handles concurrent clients via WebSocket.

### Batch Processing (Modal)

```bash
# Process a folder of images on a remote A100
modal run modal_app.py --image-folder ./office_loop --submap-size 16 --max-loops 1
```

Uploads your images, runs full SLAM, and downloads poses + dense point clouds to `./modal_results/`.

### Local Mode

```bash
# Run the streaming server locally
python -m server.app --port 5000

# Or run offline SLAM with Viser visualization (localhost:8080)
python main.py --image_folder /path/to/images --max_loops 1 --vis_map

# Quick test with bundled sample data
unzip office_loop.zip
python main.py --image_folder office_loop --max_loops 1 --vis_map
```

### Collecting Custom Data

Open Reality accepts three input types:

**Live camera** — open the sender page on any phone, no install needed.

**Recorded video** — upload directly via the UI, or extract frames manually:

```bash
ffmpeg -i /path/to/video.MOV -vf "fps=10" /path/to/frames/frame_%04d.jpg
```

**Image folder** — pass any folder of frames to `--image_folder`. Images are sorted by the numeric value in their filename. Use horizontal video for best results.

### Key Parameters

| Flag | Default | Description |
|------|---------|-------------|
| `--submap_size` | 16 | Frames per submap batch |
| `--min_disparity` | 50 | Optical flow threshold for keyframe selection |
| `--conf_threshold` | 25 | Filter bottom N% lowest-confidence points |
| `--lc_thres` | 0.95 | Loop closure similarity threshold |
| `--max_loops` | 0 | Enable loop closure (0 or 1) |
| `--vis_voxel_size` | — | Downsample point cloud for visualization |
| `--run_os` | off | Enable open-set 3D object detection |

---

## Architecture

```
Phone Camera / Video File / Image Folder
    ↓  WebSocket stream (HTTPS via Modal tunnel)
Keyframe Selection
    Lucas-Kanade optical flow — skip frames without motion
    ↓
VGGT-1B Vision Model
    Predicts dense depth + camera pose per frame
    No GPS. No depth sensor. Just pixels.
    ↓
CLIP + SAM3
    CLIP scores every submap against target queries
    SAM3 segments matches → projects to 3D bounding box
    ↓
GTSAM Pose Graph
    SL(4) manifold optimization
    Loop closure keeps the global map consistent
    ↓
Spatial Agent (Claude / Gemini)
    Interprets user intent, plans searches,
    answers spatial questions with real geometry
```

### Key Components

| Component | Role |
|-----------|------|
| **Solver** (`solver.py`) | Central coordinator — owns the map, pose graph, retrieval system, and viewer |
| **StreamingSLAM** (`server/streaming_slam.py`) | Wraps Solver for frame-by-frame WebSocket streaming |
| **PoseGraph** (`graph.py`) | GTSAM SL(4) manifold optimization with inter-submap and loop closure constraints |
| **ObjectDetector** (`object_detector.py`) | PE-Core CLIP + SAM3 for open-set 3D bounding box detection |
| **SpatialAgent** (`server/spatial_agent.py`) | LLM-powered agent with SLAM-backed tools for spatial reasoning |
| **ImageRetrieval** (`loop_closure.py`) | DINO-Salad descriptors for loop closure detection |
| **Frontend** (`server/webserver/`) | Vite + TypeScript + Three.js — 3D viewer, camera sender, plan UI, summary page |

### Frontend Pages

| Page | Purpose |
|------|---------|
| `index.html` | Live 3D SLAM viewer with point clouds and detections |
| `sender.html` | Camera/video input — streams frames to the server |
| `plan.html` | Agent plan visualization — mission planning UI |
| `summary.html` | Detection summary — 2D floorplan + 3D overview + spatial Q&A |

---

## Built With

Python · **Modal** · PyTorch · VGGT-1B · GTSAM · CLIP · SAM3 · DINO-Salad · Flask · Socket.IO · Three.js · Vite · TypeScript · Claude · Gemini

---

## Acknowledgements

Open Reality builds on [VGGT-SLAM 2.0](https://arxiv.org/abs/2601.19887) by [Dominic Maggio](https://dominic101.github.io/DominicMaggio/) and [Luca Carlone](https://lucacarlone.mit.edu/) at MIT SPARK Lab. We extend their research-grade dense SLAM system into a cloud-native, agentic spatial AI platform — deployed on Modal, accessible from any phone, and powered by autonomous spatial reasoning.

---

<div align="center">

**Open Reality** · HackIllinois 2026 · Modal Track

*Spatial intelligence for anyone, anywhere.*

</div>
