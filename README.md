# Industrial Video Analytics & Safety Rule Engine

An end-to-end computer vision project for **industrial safety monitoring and rule-based video analytics**.

Instead of stopping at object detection, this project combines **YOLO detection, ByteTrack multi-object tracking, spatial/temporal rule logic, automated violation detection, alert generation, and structured event logging**.

The focus of the project is the **rule-engine layer** that converts tracked objects into meaningful safety events.

---

## Project Overview

Traditional object detection systems answer:

> What objects are visible in the scene?

This project goes further and answers:

> What are those objects doing, where are they located, how long have they been there, and does their behavior violate a configured safety rule?

The pipeline supports configurable industrial video analytics rules using persistent Track IDs, polygon ROIs, timers, virtual lines, object counts, and violation persistence.

---

## System Architecture

```text
Video Input
    ↓
YOLO Detection
    ↓
ByteTrack Multi-Object Tracking
    ↓
Rule Evaluation Engine
    ↓
Violation Validation
    ↓
Alert Generation + Event Logging + Video Analytics
```

The final architecture diagram is available in:

```text
industrial_safety_rule_engine_architecture_clean.png
```

---

## Implemented Rule Modules

| Rule Module | Rule Logic | Confirmed Events |
|---|---|---:|
| Person Restricted-Zone Intrusion | Person enters a configured polygon ROI and remains inside for the configured persistence period | 2 |
| MHE Restricted-Zone Intrusion | Forklift/MHE enters a prohibited polygon ROI | 1 |
| Dwell-Time / Object-Stay | Tracked person remains inside a configured ROI for at least 5 seconds | 2 |
| Directional Line Crossing | Person crosses a virtual line in the monitored RIGHT → LEFT direction | 2 |
| Object Count / Occupancy Limit | Number of persons inside ROI exceeds configured limit of 4 with persistence validation | 1 |

**Total confirmed safety events: 8**

---

## Key Features

- Multi-model YOLO inference
- Persistent object tracking with ByteTrack
- Track-ID-aware rule evaluation
- Polygon ROI monitoring
- Person and MHE restricted-zone rules
- Dwell-time / object-stay timers
- Direction-aware virtual line crossing
- Class-specific object counting
- Occupancy threshold monitoring
- Temporal persistence to reduce transient false alerts
- Active and cumulative violation counters
- Automated violation snapshots
- Per-rule CSV event logs
- Consolidated master safety event log
- Annotated H.264 portfolio demo videos

---

## Rule 1 — Person Restricted-Zone Intrusion

A configurable polygon defines a restricted area.

For each tracked person:

```text
Person Detected
      ↓
Track ID Assigned
      ↓
Bottom-Center Inside ROI?
      ↓
Persistence Threshold Reached?
      ↓
Restricted-Zone Violation
```

The final demo generated:

- 2 confirmed intrusion events
- Track-specific violation overlays
- Alert snapshots
- Structured CSV events

---

## Rule 2 — MHE Restricted-Zone Intrusion

The same ROI-based rule-engine concept is applied independently to industrial vehicles.

The system tracks MHE/forklift detections and evaluates whether the tracked object enters a prohibited vehicle zone.

Final result:

```text
Confirmed MHE intrusion events: 1
```

This demonstrates that the rule engine is not limited to person-based analytics.

---

## Rule 3 — Dwell-Time / Object-Stay

This rule detects when a tracked person remains inside a configured ROI longer than an allowed duration.

Configured threshold:

```text
Dwell Threshold = 5 seconds
```

The system maintains time independently for each Track ID.

Example logic:

```text
Person enters ROI
      ↓
Start Track-ID timer
      ↓
Still inside?
      ↓
Timer ≥ 5 seconds
      ↓
Dwell-Time Violation
```

Final result:

```text
Confirmed dwell violations: 2
```

---

## Rule 4 — Directional Line Crossing

A virtual line is placed inside the scene and movement is evaluated using tracked object coordinates.

Configured monitored direction:

```text
RIGHT → LEFT
```

A small deadband/hysteresis region is used around the virtual line to reduce duplicate or jitter-based triggers.

Final events:

```text
Person #1 → RIGHT_TO_LEFT → 3.60 s
Person #4 → RIGHT_TO_LEFT → 5.03 s
```

**Total confirmed directional crossings: 2**

---

## Rule 5 — Object Count / Occupancy Limit

The system counts only target objects whose reference point is inside a configured ROI.

Configuration:

```text
Target Class       : person
Occupancy Limit    : 4
Persistence        : 1 second
Observed Count     : 4–6 persons
```

Validation statistics:

```text
Minimum Count      : 4
Maximum Count      : 6
Average Count      : 4.43
Frames Over Limit  : 170
```

A violation is generated only when the count remains above the configured limit for the persistence period.

Final confirmed occupancy events:

```text
1
```

---

## Active vs Total Violations

The project separates current system state from cumulative event history.

```text
Active Violations
= violations currently active in the scene

Total Violations
= confirmed violation events generated during the video
```

For example:

```text
Person Count: 5
Limit: 4
Excess: 1
Active Violations: 1
Total Violations: 1
```

When the count returns to normal:

```text
Active Violations: 0
Total Violations: 1
```

This makes the output suitable for live monitoring interfaces.

---

## Automated Event Logging

Each rule produces structured event records containing rule-specific information such as:

```text
event_id
timestamp
track_id
object class
confidence
direction
dwell duration
current object count
configured threshold
snapshot path
```

Individual rule logs are combined into:

```text
master_safety_event_log.csv
```

Final master log:

```text
Rules represented       : 5
Confirmed safety events : 8
```

---

## Alert Evidence

Every confirmed event can generate an automated image snapshot.

Final verification:

```text
Person Restricted Zone     : 2 snapshots
MHE Restricted Zone        : 1 snapshot
Dwell-Time                 : 2 snapshots
Directional Line Crossing  : 2 snapshots
Object Count               : 1 snapshot

Total                      : 8 snapshots
```

---

## Final Portfolio Verification

| Metric | Result |
|---|---:|
| Final Rule Modules | 5 |
| Successfully Implemented | 5 / 5 |
| Portfolio Artifacts Verified | 12 / 12 |
| Confirmed Safety Events | 8 |
| Alert Snapshots | 8 |
| Rules in Master Event Log | 5 |

**Final artifact verification: PASSED**

---

## Output Videos

The project generates separate annotated H.264 videos for each final rule:

```text
person_restricted_zone_demo.mp4
mhe_restricted_zone_intrusion_demo.mp4
dwell_time_violation_demo.mp4
directional_line_crossing_demo.mp4
object_count_demo.mp4
```

The videos contain rule-specific overlays, tracking IDs, ROI/line visualization, and violation state information.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| Ultralytics YOLO | Object detection |
| ByteTrack | Persistent multi-object tracking |
| OpenCV | Video processing and visualization |
| NumPy | Coordinate and geometry operations |
| Pandas | Event logging and result analysis |
| Matplotlib | Validation and architecture visualization |
| FFmpeg | H.264 video generation |

---

## Project Structure

```text
industrial_safety_engine/
│
├── configs/
│   └── bytetrack_portfolio.yaml
│
├── outputs/
│   │
│   ├── videos/
│   │   ├── person_restricted_zone_demo.mp4
│   │   ├── mhe_restricted_zone_intrusion_demo.mp4
│   │   ├── dwell_time_violation_demo.mp4
│   │   ├── directional_line_crossing_demo.mp4
│   │   └── object_count_demo.mp4
│   │
│   ├── alerts/
│   │   ├── restricted_zone/
│   │   ├── mhe_restricted_zone/
│   │   ├── dwell_time/
│   │   ├── directional_line_crossing/
│   │   └── object_count/
│   │
│   ├── logs/
│   │   ├── restricted_zone_events.csv
│   │   ├── mhe_restricted_zone_events.csv
│   │   ├── dwell_time_events.csv
│   │   ├── directional_line_crossing_events.csv
│   │   ├── object_count_events.csv
│   │   └── master_safety_event_log.csv
│   │
│   ├── final_rule_results_summary.csv
│   └── industrial_safety_rule_engine_architecture_clean.png
│
└── notebook/
    └── industrial_video_analytics_safety_rule_engine.ipynb
```

---

## Why This Project Matters

Object detection alone is usually not enough for real industrial video analytics.

Real deployments often require logic such as:

```text
Is a person inside a restricted area?

How long has the person remained there?

Did they cross a boundary in a prohibited direction?

How many people are currently inside an area?

Did an industrial vehicle enter a restricted zone?

Has the condition persisted long enough to generate a reliable alert?
```

This project demonstrates how **object detection + persistent tracking + configurable rule logic** can be combined into an event-driven industrial monitoring pipeline.

---

## Extensibility

The architecture can be extended with additional rule modules such as:

- PPE compliance
- Person–vehicle proximity
- Speed thresholds
- Entry/exit counting
- Vehicle dwell monitoring
- Queue monitoring
- Custom client-defined ROI rules
- Multi-condition AND/OR rule groups
- Real-time API or dashboard integration

The rule engine is designed so that new business logic can be added on top of the same detection and tracking pipeline.

---

## Final Status

**Project completed successfully.**

```text
5 final rule modules
8 verified safety events
8 automated alert snapshots
12/12 final artifacts verified
```

The project demonstrates an end-to-end workflow from **video inference to actionable safety-event generation**.
