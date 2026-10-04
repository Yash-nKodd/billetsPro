# BilletVision AI — Automated Steel Billet Inspection & Traceability System

**BilletVision AI** is an industrial control-room web application designed for automated steel billet dimensional metrology, high-temperature OCR heat/batch identification, 6-zone multi-surface defect classification, deterministic tolerance evaluation (`PASS` / `FAIL` / `REVIEW`), and end-to-end metallurgical traceability.

---

## Architecture Overview

```
                      +---------------------------------------------+
                      |             BilletVision AI UI             |
                      |   React 19 + TypeScript + Vite + Tailwind   |
                      |     Industrial Dark Graphite Control Room   |
                      +----------------------+----------------------+
                                             |
                                  HTTP REST / API Proxy
                                             |
                                             v
                      +---------------------------------------------+
                      |         FastAPI Python Backend              |
                      |     Uvicorn Asynchronous Service (Py3.14)   |
                      +----------------------+----------------------+
                                             |
         +-----------------+-----------------+-----------------+
         |                 |                 |                 |
         v                 v                 v                 v
+----------------+ +----------------+ +----------------+ +----------------+
| Quality Engine | | Vision Adapter | |   OCR Service  | | Export Engine  |
| - Dimensions   | | - OpenCV RGB   | | - PaddleOCR    | | - Pandas CSV   |
| - Tolerances   | | - YOLO Billet  | | - Heat/Batch   | | - OpenPyXL     |
| - Decisions    | | - Defect Model | | - Confidence   | |   9-Sheet XLSX |
+----------------+ +----------------+ +----------------+ +----------------+
         |                 |                 |                 |
         +-----------------+-----------------+-----------------+
                                             |
                                             v
                      +---------------------------------------------+
                      |       SQLite Local Persistence (WAL)        |
                      |    16 Normalized Tables - Repository Layer  |
                      |  (Inspections, Config, Audits, Cameras,     |
                      |   Surfaces, Measurements, Search History)   |
                      +---------------------------------------------+
```

### Key Engineering Principles
1. **Honest AI & Hardware Telemetry:** The application never fabricates detections or simulates camera connectivity when hardware is absent. If models or cameras are not connected, the UI explicitly flags the components as unavailable/disconnected and switches to **Demo Mode**.
2. **Deterministic Quality Rules:** Tolerance decisions (`PASS`, `FAIL`, `REVIEW`) are evaluated by the deterministic Quality Decision Engine against calibrated millimeters and verified thresholds.
3. **Bottom Surface Integrity Rule:** An unobserved bottom surface is never classified as PASS. Without an active conveyor sensor or optical mirror assembly, the billet is held for REVIEW.
4. **Safety-First Reject Handling:** The software generates inspection records, alerts, and audit logs. It does **not** autonomously fire industrial mechanical reject kickers without hardwired safety interlocks.
5. **Duplicate Suppression:** Billet-crossing inspection zones enforce configurable duplicate suppression windows to avoid double-counting the same billet on the roll table.

---

## All 16 Navigation Workstation Modules

| # | Navigation Item | Route | Operational Capability |
| :-: | :--- | :--- | :--- |
| **1** | **Overview Dashboard** | `/` | Real-time counts (Inspected, PASS, FAIL, REVIEW), Pass Rate %, Latency, FPS, 7-day inspection trends (Recharts), defect distribution, recent inspections, and active alerts. |
| **2** | **Live Inspection** | `/live` | Real-time video/camera canvas with bounding box overlays, toolbar (Start/Pause/Stop, Camera select, Video/Image upload), 3 demo scenario triggers, and right-hand billet inspection panel. |
| **3** | **Camera Management** | `/cameras` | USB & RTSP camera inputs, connection status probe (`CONNECTED`, `DISCONNECTED`, `PROCESSING`, `ERROR`), 6-surface assignment, and FPS/shutter telemetry. |
| **4** | **Image Inspection** | `/image-inspection` | Dedicated image upload workstation (JPG/PNG/WEBP), OpenCV contour isolation, calibrated calipers, OCR heat extraction, defect bounding boxes, and image export. |
| **5** | **Video Inspection** | `/video-inspection` | Video inspection pipeline with tracking ID persistence, virtual inspection-zone line crossing trigger, duplicate frame suppression, and snapshot generation. |
| **6** | **Inspection History** | `/history` | Searchable and filterable data table (by Date, Status, Billet ID, Heat #), detail inspector, and on-demand CSV & XLSX export. |
| **7** | **Digital Billet Passport** | `/passport` | Complete metallurgical identity record for each billet: dimensions vs tolerance bands, raw OCR text, confidence gauge, detected defect locations, 8-step timeline, and audit logs. |
| **8** | **Measurements** | `/measurements` | Calibrated dimensional metrology table: Total length, width, height, rhomboidity, camber deviation, tolerance bounds, deviations, uncertainty budget, and 3D depth notice. |
| **9** | **Surface Inspection** | `/surfaces` | 6-Zone 360° inspection layout (TOP, BOTTOM, FRONT, BACK, LEFT, RIGHT), surface-specific camera status, defect counts, and critical bottom sensor verification. |
| **10** | **OCR & Identification** | `/ocr` | PaddleOCR character recognition, raw vs cleaned string parsing, confidence score gauge, and operator correction modal with persistent audit logging. |
| **11** | **Quality Analytics** | `/analytics` | 30-day quality trends, dimensions distribution, defect breakdown, batch-level comparisons, and statistical drift alarms when deviation exceeds tolerance limits. |
| **12** | **Alerts & Review Queue**| `/alerts` | Urgent out-of-tolerance and low-confidence alarms with operator acknowledgement, manual REVIEW resolution, and ID correction modal. |
| **13** | **Calibration** | `/calibration` | Sensor geometry calibration tool: reference length (mm), pixel distance (px), computed scale factor ($mm/px$), measurement plane selection, and target tolerances. |
| **14** | **Reports & Excel Export** | `/reports` | Comprehensive report compiler generating multi-sheet Excel workbooks (all 9 required sheets) and audit-ready CSV exports. |
| **15** | **Search History** | `/search-history` | Persistent log of operator queries, applied filters, and result counts with one-click **Reopen Search** and query clear capability. |
| **16** | **Settings** | `/settings` | Configuration for inspection rules, target dimensions, OCR confidence thresholds, duplicate suppression window, and system health status. |

---

## Excel Workbook Export (9 Sheets)

When exporting to Excel (`.xlsx`), the engine automatically builds and formats all 9 sheets via `pandas` and `openpyxl`:
- **Sheet 1: Inspection Summary** (All inspected billets, grades, timestamps, decisions)
- **Sheet 2: Measurements** (Length, width, height, deviations, tolerances, uncertainties)
- **Sheet 3: Surface Inspection** (Top, Bottom, Front, Back, Left, Right coverage)
- **Sheet 4: OCR Results** (Raw stamp text, cleaned heat numbers, confidence scores)
- **Sheet 5: Defects** (Defect classes, surface assignments, bounding boxes)
- **Sheet 6: Alerts** (Critical & warning notifications, acknowledgement state)
- **Sheet 7: Review & Corrections** (Audit record of operator modifications)
- **Sheet 8: Search History** (Logged user queries and filter criteria)
- **Sheet 9: Audit Log** (Timestamped system and configuration event history)

---

## Getting Started

### Prerequisites
- **Python:** 3.10+ (tested on Python 3.14)
- **Node.js:** v18+ (tested on Node v24 with Vite 8)

---

### Backend Service

1. Navigate to backend:
   ```bash
   cd backend
   ```
2. Run automated test suite:
   ```bash
   python -m pytest tests/ -v
   ```
   *30 passed tests covering metrology, tolerances, 9-sheet XLSX exports, search history, and persistence.*

3. Start FastAPI server:
   ```bash
   python -m uvicorn main:app --host 127.0.0.1 --port 8000
   ```
   *Available at `http://127.0.0.1:8000` with Swagger UI at `http://127.0.0.1:8000/docs`.*

---

### Frontend Dashboard

1. Navigate to frontend:
   ```bash
   cd frontend
   ```
2. Build production bundle:
   ```bash
   npm run build
   ```
3. Start Vite dev server:
   ```bash
   npm run dev
   ```
   *Dashboard available at `http://127.0.0.1:5173`.*

---

## Operating in Demo Mode

When physical RGB cameras or YOLO model weights (`.pt`) are not connected:
1. The Top Bar displays the amber **`DEMO MODE`** badge and indicates component availability accurately.
2. In **Live Inspection**, three deterministic industrial scenarios can be triggered immediately:
   - **`Pass Scenario`**: Billet within target ($6002 \times 149\,\text{mm}$), clear heat number `HN-2024-0847`, 95% OCR confidence $\rightarrow$ Status: `PASS`.
   - **`Fail Scenario`**: Length exceeds tolerance ($6078\,\text{mm}$, $+28\,\text{mm}$ deviation), surface crack detected $\rightarrow$ Status: `FAIL`.
   - **`Review Scenario`**: Unclear stamp `HN-2024-08??`, OCR confidence 0.42 ($<0.70$ threshold) $\rightarrow$ Status: `REVIEW`.
3. Generated records are persisted directly to SQLite and instantly populate all 16 workstation modules.
#   b i l l e t s P r o  
 