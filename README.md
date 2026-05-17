# 🏀 Multi-Object Detection and Persistent ID Tracking — Basketball

A computer vision pipeline that detects and tracks multiple players in a public basketball video using YOLOv8 and ByteTrack, with trajectory visualization and movement analytics.

---

## 📽️ Video Source

- **Source:** Pexels (free stock footage, no copyright restrictions)
- **File:** `19570050-uhd_3840_2160_24fps.mp4`
- **Duration:** ~17 seconds
- **Resolution:** 3840x2160 (4K UHD) downscaled to 384x640 for inference
- **FPS:** 24
- **Total Frames:** 424
- **Setting:** Outdoor night basketball court

---

## 🎯 Results Summary

| Metric | Value |
|---|---|
| Total frames processed | 424 |
| Unique player IDs assigned | 43 |
| Average players per frame | 8.2 |
| Max players in a single frame | 12 |
| Min players in a single frame | 4 |
| Avg inference speed | ~11ms per frame |
| Detection confidence threshold | 0.30 |

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| Object Detection | YOLOv8n (Ultralytics) |
| Multi-Object Tracking | ByteTrack (via Ultralytics) |
| Video I/O | OpenCV |
| Visualization | Matplotlib |
| Environment | Google Colab (T4 GPU) |
| Language | Python 3.12 |

---

## 📁 Repository Structure

```
basketball-tracking/
│
├── basketball_tracking.ipynb     # Main Colab notebook
├── README.md                     # This file
├── technical_report.md           # Technical report (1-2 pages)
│
├── screenshots/
│   ├── frame_50.jpg              # Sample detection frame
│   ├── frame_100.jpg
│   ├── frame_200.jpg
│   ├── frame_300.jpg
│   └── frame_400.jpg
│
├── outputs/
│   ├── trajectories.jpg          # Player movement trajectory visualization
│   ├── player_count_chart.png    # Player count over time chart
│   └── annotated_output.avi      # Full annotated output video
│
└── sample_input/
    └── 19570050-uhd_3840_2160_24fps.mp4   # Original input video
```

---

## ⚙️ Installation & Setup

### Requirements
- Python 3.8+
- Google Colab (recommended) or local environment with GPU

### Install Dependencies

```bash
pip install ultralytics opencv-python-headless matplotlib
```

### Run in Google Colab (Recommended)

1. Open `basketball_tracking.ipynb` in Google Colab
2. Set runtime to **T4 GPU**: Runtime → Change runtime type → T4 GPU
3. Upload your input video when prompted
4. Run all cells in order

### Run Locally

```bash
git clone https://github.com/YOUR_USERNAME/basketball-tracking
cd basketball-tracking
pip install ultralytics opencv-python matplotlib
# Place your video in the project root
python tracking_pipeline.py
```

---

## 🔄 Pipeline Overview

```
Input Video
    │
    ▼
YOLOv8n Detection (person class only)
    │
    ▼
ByteTrack ID Assignment
    │
    ▼
Per-Frame Bounding Box + ID Annotation
    │
    ├──► Annotated Output Video
    ├──► Screenshots (5 sample frames)
    ├──► Trajectory Visualization
    └──► Player Count Over Time Chart
```

---

## 📊 Visualizations

### 1. Annotated Video
Each player is marked with a bounding box and a persistent unique ID (e.g., `id:1 person 0.82`). Confidence score is shown alongside each detection.

### 2. Trajectory Map
All player movement paths are overlaid on the first frame of the video, with each player assigned a unique color. Shows spatial coverage of players across the entire 17-second clip.

### 3. Player Count Chart
A time-series chart showing how many players were detected per frame, with the average (8.2) marked as a reference line.

---

## ⚠️ Assumptions

- Only the `person` class (class 0) is detected — the basketball itself is not tracked
- The video is processed at full frame rate (every frame, no skipping)
- ByteTrack default parameters are used without additional tuning
- The model was not fine-tuned on basketball-specific data

---

## 🔍 Limitations

- **ID reassignment:** 43 unique IDs were assigned across ~10 actual players due to players leaving and re-entering the frame, causing ByteTrack to assign new IDs
- **Nighttime lighting:** Low-light conditions reduce detection confidence for players further from the floodlight
- **Occlusion:** When players overlap, the tracker occasionally loses and re-acquires the same player with a new ID
- **Label overlap:** At high player density, bounding box labels overlap and become hard to read
- **No ball tracking:** The basketball is not detected or tracked in this pipeline

---

## 🚀 Possible Improvements

- Use a larger YOLOv8 model (yolov8m or yolov8l) for better accuracy
- Fine-tune on sports-specific datasets (e.g., SoccerNet, SportsMOT)
- Add ReID (re-identification) module to maintain consistent IDs across re-entries
- Implement team clustering by jersey color
- Add ball detection as a separate class
- Use StrongSORT instead of ByteTrack for better long-term ID consistency

---

## 🤖 AI Tools Used

- **Claude (Anthropic)** — assisted with pipeline design, code structure, and documentation
- **Ultralytics YOLOv8** — pre-trained detection and tracking model
- All detections, outputs, and visualizations were generated by running the actual pipeline on the real video

---

## 📄 License

This project uses:
- YOLOv8 under the [AGPL-3.0 License](https://github.com/ultralytics/ultralytics/blob/main/LICENSE)
- Input video from [Pexels](https://www.pexels.com/license/) (free to use)
