# RailSafe System: Complete End-to-End Architecture & Technical Documentation

This document provides a **comprehensive, technical explanation** of the RailSafe System, covering every feature, the complete tech stack, and detailed implementation of each component.

---

## 🏗️ System Architecture Overview

The RailSafe System is a full-stack anomaly detection platform combining **vibration sensors**, **computer vision**, **AI fusion logic**, and **LLM-powered explanations** to monitor railway infrastructure in real-time.

```mermaid
graph TB
    subgraph "Data Ingestion Layer"
        ACC[3-Axis Accelerometer<br/>100Hz] -->|acc_x, acc_y, acc_z| VP[Vibration Pipeline<br/>Python]
        CCTV[CCTV Camera<br/>15 FPS] -->|Video Frames| RFB[Rolling Frame Buffer<br/>30s History]
    end

    subgraph "Vibration Analysis - Python"
        VP -->|Butterworth Filter| SP[Signal Processor]
        SP -->|2s Windows| FE[Feature Extractor<br/>RMS, Crest, HF Energy]
        FE --> AD[Anomaly Detector<br/>Threshold: Crest > 3.8]
        AD --> ESS[Edge Spike Suppressor]
        ESS -->|Confirmed| IM[Incident Manager]
        IM -->|ALERT_START Event| VTM[Vision Trigger]
    end

    subgraph "Vision Analysis - YOLOv8"
        VTM -->|Timestamp T| RFB
        RFB -->|Extract T-5s to T+5s| YIE[YOLO Inference Engine<br/>person, animal, train, track]
        YIE -->|Bounding Boxes & Masks| SOT[Object Tracker<br/>Centroid-based]
        SOT --> SA[Spatial Analyzer<br/>On-Track Detection via IoU]
        SA --> VCB[Vision Context Builder<br/>JSON Output]
    end

    subgraph "Fusion & Intelligence"
        IM -->|Vibration Summary| FE_ENG[Fusion Rule Engine]
        VCB -->|Vision Summary| FE_ENG
        FE_ENG -->|Deterministic Rules| CLASS[Incident Classifier]
        CLASS -->|Fused JSON| LLM_GEN[LLM Summary Generator<br/>Gemini Flash]
        LLM_GEN --> DB[(PostgreSQL<br/>Prisma ORM)]
    end

    subgraph "Backend API - Node.js/Express"
        DB --> API[REST API + Socket.IO]
        API -->|Real-time Events| FE_APP[React Frontend]
    end

    subgraph "Frontend - React Application"
        FE_APP --> DASH[Dashboard<br/>Stats & Alerts]
        FE_APP --> RTM[Real-Time Monitor<br/>Live Telemetry]
        FE_APP --> MAP[Network Map<br/>Train Schedules]
        FE_APP --> DRONE[Drone Inspection<br/>UAV Reports]
        FE_APP --> ISSUES[Issue Management<br/>Alert Triage]
        FE_APP --> BLOCKS[Block List<br/>Infrastructure View]
        FE_APP --> CHAT[AI Chatbot<br/>Incident Q&A]
    end

    subgraph "Feedback Loop"
        ISSUES -->|Dismiss as False Positive| FBL[Feedback Learner]
        FBL -->|Update Context Penalty| FE_ENG
    end

    CHAT -->|User Questions| CHATBOT[Python Chatbot<br/>Gemini LLM]
    CHATBOT -->|Context from DB| DB
```

---

## 📚 Technology Stack

### **Frontend (Client)**
| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 19.2.0 | UI Framework |
| **Vite** | 7.2.4 | Build Tool & Dev Server |
| **TailwindCSS** | 3.4.19 | Styling Framework |
| **React Router** | 7.11.0 | Client-Side Routing |
| **Socket.IO Client** | 4.8.3 | Real-Time WebSocket Communication |
| **Axios** | 1.13.2 | HTTP Client for API Requests |
| **Recharts** | 3.6.0 | Real-Time Data Visualization (Charts) |
| **Leaflet** | 1.9.4 | Interactive Maps |
| **React Leaflet** | 5.0.0 | React Bindings for Leaflet |
| **Framer Motion** | 12.23.26 | Animations & Transitions |
| **Lucide React** | 0.562.0 | Icon Library |
| **jsPDF** | 4.0.0 | Client-Side PDF Generation |
| **React Markdown** | 10.1.0 | Markdown Rendering for Chatbot |

### **Backend (Server)**
| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | - | Runtime Environment |
| **Express** | 5.2.1 | Web Server Framework |
| **Prisma** | 5.22.0 | ORM for Database Management |
| **PostgreSQL** | - | Relational Database |
| **Socket.IO** | 4.8.3 | Real-Time Bidirectional Communication |
| **CORS** | 2.8.5 | Cross-Origin Resource Sharing |
| **dotenv** | 17.2.3 | Environment Variable Management |

### **Python (ML/AI Layer)**
| Library | Purpose |
|---------|---------|
| **NumPy** | Numerical Computing for Signals |
| **Pandas** | Data Manipulation (CSV Loading) |
| **SciPy** | Signal Filtering (Butterworth), FFT |
| **OpenCV (cv2)** | Video Frame Processing |
| **Ultralytics (YOLOv8)** | Object Detection & Segmentation |
| **LangChain** | LLM Framework for Chatbot |
| **Google Generative AI** | Gemini Flash Lite API |
| **SQLite3** | Chat History Storage |

---

## 🔄 Complete System Flow (Phase-by-Phase)

### **Phase 1: Data Ingestion**

#### 1.1 Vibration Sensor Stream
- **Hardware**: 3-axis accelerometer mounted on railway tracks
- **Sampling Rate**: 100 Hz (100 samples/second)
- **Data Format**: `{ timestamp, acc_x, acc_y, acc_z }`
- **Transport**: Data is ingested by Python script (`realtime_bridge.py` or `realtime_bridge_full.py`)

#### 1.2 CCTV Video Stream
- **Hardware**: Fixed CCTV camera overlooking track section
- **Frame Rate**: 15 FPS
- **Processing**: Frames are continuously buffered in a **Rolling Frame Buffer** (stores last 30 seconds)
- **Purpose**: Provides "time-travel" capability to analyze video from *before* an alert is triggered

---

### **Phase 2: Vibration Analysis (The Trigger)**
**File**: `vibration_pipeline.py`  
**Tech Stack**: Python, NumPy, SciPy

#### 2.1 Signal Processing (`StreamingSignalProcessor`)
```python
# Butterworth High-Pass Filter (removes DC offset & gravity)
cutoff = 0.5 Hz
filter_order = 4
b, a = scipy.signal.butter(4, 0.5/nyquist, btype='high')

# Compute magnitude from 3-axis data
magnitude = sqrt(x² + y² + z²)
```

#### 2.2 Feature Extraction (`FeatureExtractor`)
Every **2-second sliding window** (200 samples), the system extracts:

| Feature | Formula | Meaning |
|---------|---------|---------|
| **RMS** | `sqrt(mean(signal²))` | Overall vibration energy |
| **Crest Factor** | `max(abs(signal)) / RMS` | Spikiness (high = impacts) |
| **HF Energy** | `sum(FFT[5-20Hz]²)` | High-frequency content (scraping/metal) |
| **Peak Count** | `count(signal > 2*RMS)` | Number of sharp peaks |
| **Variance** | `var(signal)` | Signal irregularity |

#### 2.3 Anomaly Detection Logic (`AnomalyDetector`)
```python
# Primary Condition
is_anomaly = (crest_factor > 3.8) AND 
             (hf_energy > threshold OR peak_count > 10 OR high_variance)
```

#### 2.4 Edge Spike Suppression (`EdgeSpikeSupressor`)
- **Problem**: Legitimate signals often have "edge spikes" at start/end
- **Solution**: Requires **2 consecutive anomalous windows** before confirming
- **RMS Slope Check**: Rejects sudden RMS jumps (> 0.15) if not sustained

#### 2.5 Maintenance Schedule Integration (`MaintenanceSchedule`)
- **Input**: CSV file with timestamps of scheduled maintenance
- **Logic**: If current timestamp matches maintenance window → **Suppress Alert**
- **Purpose**: Prevents false alarms during authorized track work

#### 2.6 Incident Management (`IncidentManager`)
- Tracks active incidents with unique IDs (`INC_0001`)
- Emits **`ALERT_START`** event when incident is confirmed (after 2+ windows)
- Includes severity: `HIGH` (Crest > 5.0), `MEDIUM` (Crest > 4.2), `LOW`

---

### **Phase 3: Event-Triggered Vision Analysis**
**Files**: `vision_pipeline_part1.py`, `vision_pipeline_part2.py`  
**Tech Stack**: Python, OpenCV, YOLOv8 (Ultralytics)

#### 3.1 Vision Triggering (`VisionTriggerManager`)
When `ALERT_START` event is received:
1. **Extract Event Timestamp** (T)
2. Query `RollingFrameBuffer` for frames in window `[T - 5s, T + 5s]`
3. Result: ~150 frames (15 FPS × 10 seconds)

#### 3.2 YOLO Inference (`YOLOInferenceEngine`)
**Model**: YOLOv8 Segmentation (custom-trained on railway imagery)

**Classes Detected**:
- `person` (workers, trespassers)
- `animal` (cattle, wildlife)
- `train` (moving trains)
- `rail_track` (the track itself, used as reference)

**Output per Frame**:
```python
{
  frame_id: 123,
  class_name: "person",
  confidence: 0.94,
  bbox: (x1, y1, x2, y2),  # Bounding box
  mask: np.array([...]),    # Segmentation mask
  mask_area: 12345.0        # Pixel count
}
```

#### 3.3 Object Tracking (`SimpleObjectTracker`)
- **Algorithm**: Centroid-based tracking across consecutive frames
- **Tracking ID**: Assigns consistent IDs to objects (e.g., `PERSON_001`)
- **Purpose**: Calculate dwell time, velocity, motion state

#### 3.4 Spatial Analysis (`SpatialAnalyzer`)
Determines relationship between detected objects and rail track:

**On-Track Detection**:
```python
track_mask = detections['rail_track'].mask
object_mask = detections['person'].mask

# Calculate Intersection over Union (IoU)
overlap_ratio = intersection(track_mask, object_mask) / object_mask_area

if overlap_ratio > 0.10:
    status = "ON_TRACK"  # Critical!
elif distance_to_track < 2.0m:
    status = "NEAR_TRACK"  # Warning
else:
    status = "OFF_TRACK"  # Safe
```

**Motion State**:
- `MOVING`: Average velocity > threshold
- `STATIONARY`: Low velocity (e.g., person standing on track)

#### 3.5 Vision Context Builder (`VisionContextBuilder`)
Aggregates all vision data into `vision_context.json`:
```json
{
  "objects_detected": ["person", "rail_track"],
  "tracks": [
    {
      "track_id": "PERSON_001",
      "on_track": true,
      "dwell_time_s": 3.2,
      "motion_state": "STATIONARY",
      "confidence_avg": 0.92
    }
  ]
}
```

---

### **Phase 4: Fusion Engine (AI Decision Making)**
**File**: `fusion_engine.py`  
**Tech Stack**: Python, Deterministic Rules

#### 4.1 Fusion Rules (`FusionRuleEngine`)
The system applies **hierarchical deterministic rules**:

| Priority | Rule Name | Vibration Condition | Vision Condition | Classification | Risk Level |
|----------|-----------|---------------------|------------------|----------------|------------|
| 1 | **Tampering Detection** | High Severity (Crest > 5.0) | **No Objects** | `possible_tampering` | **CRITICAL** |
| 2 | **Train Movement** | Any | Train Detected | `train_movement` | NEGLIGIBLE |
| 3 | **Human on Track** | Anomaly | Person On-Track, Dwell < 5s | `human_crossing` | MEDIUM |
| 4 | **Stationary Trespasser** | Anomaly | Person On-Track, Dwell > 5s | `human_presence` | HIGH |
| 5 | **Animal Detection** | Anomaly | Animal Detected | `animal_presence` | MEDIUM |
| 6 | **Obstruction** | Anomaly | Object Near Track | `stationary_obstruction` | MEDIUM |
| 7 | **Fallback** | Anomaly | Nothing | `unknown` | LOW |

**Why "No Objects" is CRITICAL**: 
- System detected physical impact (vibration) but cameras see nothing
- Indicates:
  - Hidden sabotage (e.g., track bolt removal)
  - Camera failure/blind spot
  - Underground equipment malfunction

#### 4.2 LLM Summary Generation
After classification, the system calls **Google Gemini Flash Lite**:

**Prompt**:
```python
prompt = f"""
You are RailSafe AI Assistant. Generate a concise incident summary.

Incident Data:
- Classification: {classification}
- Risk Level: {risk_level}
- Vibration: High crest factor ({max_crest})
- Vision: {vision_summary}

Format:
1. What happened (1 sentence)
2. Why it's risky (1 sentence)
3. Recommended action (1 sentence)
"""
```

**Output** (saved to database as `llmSummary`):
> "A critical tampering incident was detected at Block 12. High-intensity vibration with no visible objects suggests hidden sabotage or equipment failure. Immediate physical inspection recommended."

#### 4.3 Feedback Learner (`FeedbackLearner`)
**File**: `feedback_model.json`

When operator dismisses an alert as false positive:
```python
context_key = f"{block_id}_{classification}_{time_of_day}"
feedback_model[context_key]["false_positive_count"] += 1

# Apply confidence penalty in future
if false_positive_count > 3:
    confidence_penalty = 0.2  # Reduce confidence by 20%
```

**Temporal Decay**: Penalties reduce over 30 days if no new false positives occur.

---

### **Phase 5: Backend API (Node.js/Express)**
**File**: `backend/index.js`  
**Tech Stack**: Express, Prisma, Socket.IO

#### 5.1 Database Schema (Prisma)
```prisma
model AddressBlock {
  id        Int      @id @default(autoincrement())
  name      String
  status    String   // "OK", "WARNING", "CRITICAL"
  cctvUrl   String?
  userId    Int?
  issues    Issue[]
  logs      SensorLog[]
}

model Issue {
  id          Int      @id @default(autoincrement())
  blockId     Int
  type        String   // "TAMPERING_DETECTED", "HUMAN_CROSSING", etc.
  severity    String   // "LOW", "MEDIUM", "HIGH"
  status      String   // "OPEN", "RESOLVED", "FALSE_POSITIVE"
  details     String   // JSON string of fusion data
  llmSummary  String?  // LLM-generated explanation
  incidentId  String?
  timestamp   DateTime @default(now())
}

model SensorLog {
  id        Int      @id @default(autoincrement())
  blockId   Int
  type      String   // "VIBRATION"
  value     Float
  timestamp DateTime @default(now())
}
```

#### 5.2 REST API Endpoints

| Method | Endpoint | Purpose | Returns |
|--------|----------|---------|---------|
| `POST` | `/api/auth/login` | User authentication | User object (JWT in production) |
| `GET` | `/api/blocks` | Fetch all blocks with issue counts | `AddressBlock[]` |
| `GET` | `/api/blocks/:id` | Fetch single block with logs/issues | `AddressBlock` with relations |
| `GET` | `/api/issues` | Fetch all open issues | `Issue[]` sorted by timestamp |
| `POST` | `/api/issues` | Create manual issue (e.g., reschedule request) | Created `Issue` |
| `PATCH` | `/api/issues/:id` | Update issue status (resolve/dismiss) | Updated `Issue` |
| `GET` | `/api/stats` | Dashboard statistics | `{ totalBlocks, activeIssues, resolvedToday }` |
| `GET` | `/api/schedules` | Fetch train schedules | CSV-parsed schedule array |
| `POST` | `/api/chat` | Chat with incident-specific LLM | Streaming text response |
| `GET` | `/api/chat/:id/history` | Retrieve chat history | Chat messages array |

#### 5.3 Real-Time Communication (Socket.IO)
**Events Emitted to Frontend**:

| Event | Payload | Purpose |
|-------|---------|---------|
| `telemetry` | `{ acc: {x, y, z}, features: {rms, crest, ...} }` | Live vibration data for graphs |
| `alert` | `{ incident_id, severity, details }` | Basic anomaly detection alerts |
| `fusion_alert` | `{ classification, risk_level, llm_summary, ... }` | High-quality fusion alerts |

#### 5.4 Python Bridge Integration
The backend spawns Python processes for:
1. **Real-Time Monitoring** (`realtime_bridge_full.py`):
   - Streams vibration telemetry via stdout (JSON lines)
   - Node.js parses JSON and broadcasts via Socket.IO

2. **Chatbot** (`incident_chatbot.py`):
   - Receives incident context via stdin
   - Returns LLM response via stdout

---

### **Phase 6: Frontend Application (React)**
**Framework**: React 19.2.0 + Vite  
**Routing**: React Router v7  
**State Management**: React Hooks (useState, useEffect)

#### 6.1 Page Components

##### **1. Login (`Login.jsx`)**
- **Authentication**: Sends credentials to `/api/auth/login`
- **Storage**: Saves user session in `localStorage`
- **Design**: Modern glassmorphism UI with animated hero section

##### **2. Dashboard (`Dashboard.jsx`)**
- **Stats Display**: Total blocks, active issues, resolved today
- **Live Updates**: Re-fetches stats every 3 seconds
- **Charts**: Recharts library for trend visualization
- **Quick Actions**: Navigate to blocks, issues, real-time monitor
- **AI Pipeline Visualizer**: Interactive flowchart showing data flow

##### **3. Real-Time Monitor (`RealTimeMonitor.jsx`)**
- **WebSocket**: Socket.IO client receives `telemetry` events
- **Graphs**: 
  - Live accelerometer (X, Y, Z) waveform
  - Live Crest Factor chart (with 3.8 threshold line)
  - Anomaly indicators (red flash on detection)
- **Block Filter**: Dropdown to filter telemetry by specific block
- **Video Feed**: Displays CCTV stream for selected block

##### **4. Block List (`BlockList.jsx`)**
- **Table View**: All infrastructure blocks with status badges
- **Filtering**: Filter by status (OK/WARNING/CRITICAL)
- **Navigation**: Click block to view detailed logs
- **Status Colors**:
  - 🟢 OK (Green)
  - 🟡 WARNING (Amber)
  - 🔴 CRITICAL (Red)

##### **5. Issue List (`IssueList.jsx`)**
- **Alert Queue**: All open incidents sorted by timestamp
- **Triage Actions**:
  - Mark as Resolved
  - Dismiss as False Positive
  - View in Chatbot
- **LLM Summaries**: Displays AI-generated explanations
- **Severity Badges**: Color-coded by risk (HIGH/MEDIUM/LOW)

##### **6. Network Map (`MapOverview.jsx`)**
**Tech**: Leaflet.js + React Leaflet

**Features**:
- **Live Train Positions**: 
  - Fetches schedules from `/api/schedules`
  - Interpolates train position between stations using:
    - Current time vs. departure/arrival times
    - Path-following algorithm along rail line coordinates
  - Displays trains with custom icons (pulsing animation when active)
- **Station Markers**:
  - Color-coded by line (Red/Yellow/Blue)
  - Status overlays (OK/WARNING/CRITICAL)
- **Rescheduling**:
  - Click station → "Request Reschedule" button
  - Sends `POST /api/issues` with `type: RESCHEDULE_REQUEST`
  - Logs request in database for dispatch coordination
- **Network Legend**: Shows rail lines and status indicators

##### **7. Drone Inspection (`DroneInspection.jsx`)**
**Tech**: jsPDF for report generation

**Features**:
- **Inspection Gallery**: 
  - Pre-analyzed YOLO segmentation images
  - Shows detected anomalies ("Tree Branch on Track", "Drum on Track")
  - Track integrity percentage per image
- **Filtering**: Filter by Critical (< 85% integrity) or Nominal
- **Live Feed Simulation**: 
  - Modal with drone video playback
  - Tactical HUD overlay (altitude, speed, battery, GPS)
- **PDF Report Generation**:
  - Client-side PDF using jsPDF + AutoTable
  - Includes mission summary, sector logs, recommendations
  - Styled with government/railway branding

#### 6.2 Key React Hooks Usage

**Real-Time Data Updates**:
```javascript
useEffect(() => {
  const socket = io('http://localhost:3000');
  
  socket.on('telemetry', (data) => {
    setAccData(data.acc);
    if (data.features) {
      setCrestFactor(data.features.crest);
    }
  });

  return () => socket.disconnect();
}, []);
```

**Polling for Updates**:
```javascript
useEffect(() => {
  const fetchBlocks = async () => {
    const res = await api.get('/blocks');
    setBlocks(res.data);
  };
  
  fetchBlocks();
  const interval = setInterval(fetchBlocks, 3000);
  return () => clearInterval(interval);
}, []);
```

---

### **Phase 7: AI Chatbot Integration**
**File**: `backend/incident_chatbot.py`  
**Tech**: LangChain + Google Generative AI

#### 7.1 System Architecture
```python
# Chat History Database (SQLite)
chat_history.db:
  Table: messages
    - incident_id (str)
    - role (str): "user" | "assistant"
    - content (str)
    - timestamp (datetime)

# LLM Configuration
model = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash-exp",
    temperature=0.3,  # Low for factual responses
    streaming=True    # Enables word-by-word output
)
```

#### 7.2 System Prompt (RAG Context)
```python
SYSTEM_PROMPT = """
You are RailSafe Assistant, an AI expert in railway safety.

STRICT RULES:
1. Answer ONLY based on provided incident context
2. Use Markdown formatting (bold, lists, headings)
3. If asked about data not in context, say "Not available in current incident data"
4. For system overview questions, reference active_issue_count and recent incidents

Context Schema:
- classification: Incident type
- risk_level: CRITICAL/HIGH/MEDIUM/LOW
- vibration_summary: { max_crest, severity, ... }
- vision_summary: { objects_detected, tracks, ... }
- fusion_logic: { explanation, confidence, ... }
"""
```

#### 7.3 Chat Modes

**A. Incident-Specific Chat**:
- **Input**: Incident ID (e.g., `INC_0042`)
- **Context**: Loads `fused_incident.json` for that incident
- **Example Questions**:
  - "Why was this marked as critical?"
  - "What did the cameras see?"
  - "Should we stop trains?"

**B. Global System Chat**:
- **Input**: `incidentId: "global"`
- **Context**: Fetches system-wide stats from database:
  - Total alerts today
  - Active issue count
  - Recent incidents (last 10)
- **Example Questions**:
  - "What's the busiest block today?"
  - "How many tampering incidents this week?"

---

## 🔧 Technical Implementation Details

### **Data Flow Implementation**

#### **1. Vibration-to-Alert Pipeline**
```
Raw Sample (100Hz)
  ↓
Butterworth Filter (scipy.signal)
  ↓
Magnitude Calculation (numpy)
  ↓
Sliding Window Buffer (collections.deque)
  ↓
Feature Extraction (RMS, Crest, FFT)
  ↓
Anomaly Detection (threshold logic)
  ↓
Edge Suppression (consecutive window check)
  ↓
Incident Manager (state machine)
  ↓
ALERT_START Event (JSON dict)
  ↓
stdout → Node.js (JSON.parse)
  ↓
Socket.IO broadcast (io.emit)
  ↓
React Frontend (socket.on)
```

#### **2. Vision-to-Context Pipeline**
```
CCTV Frame (15 FPS)
  ↓
RollingFrameBuffer.add_frame()
  ↓
[TRIGGER] ALERT_START received
  ↓
RollingFrameBuffer.extract_window(T-5s, T+5s)
  ↓
YOLOv8.predict(frames)
  ↓
Detections with Masks (ultralytics output)
  ↓
SimpleObjectTracker.update()
  ↓
SpatialAnalyzer.analyze_tracks()
  ↓
On-Track IoU Calculation (numpy mask overlap)
  ↓
vision_context.json
  ↓
Fusion Engine
```

#### **3. Fusion-to-Database Pipeline**
```
fusion_engine.apply_rules()
  ↓
Rule Priority Matching
  ↓
LLM Summary Request (Gemini API)
  ↓
fused_incident = {
  classification,
  risk_level,
  llm_summary,
  vibration_summary,
  vision_summary
}
  ↓
stdout (JSON) → Node.js
  ↓
prisma.issue.create({
  type: classification,
  severity: risk_level,
  details: JSON.stringify(fused_incident),
  llmSummary: llm_summary
})
  ↓
Socket.IO emit('fusion_alert')
  ↓
React Dashboard Update
```

---

## 🌐 Complete Feature List

### **Core Monitoring Features**
1. ✅ **Real-Time Vibration Monitoring**: Live 100Hz accelerometer data processing
2. ✅ **CCTV Integration**: Continuous video buffering with event-triggered analysis
3. ✅ **Anomaly Detection**: Multi-threshold physics-based detection (Crest Factor, HF Energy)
4. ✅ **Object Detection**: YOLOv8 segmentation for person/animal/train/track
5. ✅ **Spatial Analysis**: On-track detection via mask IoU, distance, and motion tracking
6. ✅ **Fusion Logic**: 7-tier deterministic rule engine for incident classification
7. ✅ **LLM Explanations**: AI-generated incident summaries via Gemini Flash

### **User Interface Features**
8. ✅ **Authentication System**: Login with session management
9. ✅ **Dashboard**: Real-time stats, issue counts, trend charts
10. ✅ **Real-Time Monitor**: Live accelerometer graphs, crest factor chart, video feed
11. ✅ **Block Management**: Infrastructure list with status indicators
12. ✅ **Issue Queue**: Alert triage with resolve/dismiss actions
13. ✅ **Interactive Map**: Live train positions with schedule interpolation
14. ✅ **Train Scheduling**: CSV-based schedule loading and visualization
15. ✅ **Reschedule Requests**: Operator-initiated train rescheduling
16. ✅ **Drone Inspection**: Pre-analyzed aerial inspection gallery
17. ✅ **PDF Report Generation**: Client-side surveillance report export
18. ✅ **AI Chatbot**: Incident-specific and global system Q&A

### **Advanced Features**
19. ✅ **Edge Spike Suppression**: Filters transient false positives
20. ✅ **Maintenance Schedule Integration**: Auto-suppression during planned work
21. ✅ **Feedback Learning**: Context-aware false positive penalty system
22. ✅ **Temporal Decay**: Automatic sensitivity restoration over time
23. ✅ **Socket.IO Streaming**: Sub-second latency for alerts and telemetry
24. ✅ **Multi-Block Support**: 70+ railway block network simulation
25. ✅ **Path-Following Animation**: Trains follow actual rail line geometry on map

---

## 🔄 Summary Data Flow

**Complete End-to-End Journey**:
```
Physical Impact on Track
  ↓
3-Axis Accelerometer (100Hz)
  ↓
Python Vibration Pipeline (Filtering, Feature Extraction, Anomaly Detection)
  ↓
ALERT_START Event (Incident Manager)
  ↓
Vision Pipeline Triggered (YOLO on buffered CCTV frames)
  ↓
Spatial Analysis (On-Track Detection, Motion Tracking)
  ↓
Fusion Engine (Deterministic Rules + LLM Summary)
  ↓
PostgreSQL Database (Prisma ORM)
  ↓
Node.js REST API + Socket.IO
  ↓
React Frontend (Dashboard, Map, Chatbot)
  ↓
Operator Action (Resolve, Dismiss, Chat with AI)
  ↓
Feedback Learner (Update Future Sensitivity)
  ↓
[LOOP] Continuous Monitoring Resumes
```

---

## 📝 Development & Deployment

### **Local Development**
```bash
# Backend
cd backend
npm install
npx prisma migrate dev
npm start

# Frontend  
cd frontend
npm install
npm run dev

# Python Dependencies
pip install numpy pandas scipy opencv-python ultralytics langchain langchain-google-genai
```

### **Environment Variables**
```env
# Backend (.env)
DATABASE_URL="postgresql://user:password@localhost:5432/railsafe"
GOOGLE_API_KEY="your-gemini-api-key"
PYTHON_PATH="python"  # or full path to python.exe on Windows
```

### **Production Considerations**
- **JWT Authentication**: Replace plain password with hashed passwords + JWT tokens
- **Rate Limiting**: Add rate limiting to API endpoints
- **WebSocket Security**: Implement authentication for Socket.IO connections
- **Database Scaling**: Use connection pooling, read replicas for high traffic
- **Video Storage**: Use CDN/S3 for CCTV video storage instead of local files
- **MLOps**: Version control for YOLO model weights, A/B testing for fusion rules

---

This document covers **every component, every feature, and every technical detail** of the RailSafe System. The system represents a complete fusion of sensor physics, computer vision, AI reasoning, and modern web technologies to create an intelligent railway safety monitoring platform.
