# Scenario 7: Vulnerable Road User Detection and Automatic Emergency Braking Simulation

## 1. Project Overview

This project falls within the domain of computer vision applied to autonomous driving. It aims to detect Vulnerable Road Users (VRU) from a video stream and simulate the behavior of an Automatic Emergency Braking (AEB) system.

The solution relies on the YOLOv8n (nano version) object detection model, combined with a geometric analysis of the vehicle's driving corridor and a decision-making logic based on timers (implementation of a simplified state machine).

The system analyzes successive frames from a video, identifies relevant objects, verifies their position relative to a predefined driving corridor, and updates the system state displayed on screen.

The objective is to implement an experimental approach combining object detection, spatial analysis, and decision-making in a simplified autonomous driving environment.

## 2. Objectives

The main objectives of the project are as follows:

- Detect pedestrians and other object categories using the YOLOv8n model.
- Localize detected objects in the frames using their bounding boxes.
- Define a trapezoidal corridor approximately representing the vehicle's trajectory.
- Determine whether monitored objects are located within the studied trajectory zone.
- Trigger the emergency braking simulation when a pedestrian is detected within the corridor.
- Trigger a warning when a sports ball is detected within the corridor.
- Visualize detections, the corridor, and the system state during video processing.

## 3. Main Features

### 3.1. Object Detection

The system uses YOLOv8n, via the Ultralytics library, to detect objects present in each frame.

For each detection, the model provides:

- Bounding box coordinates (`box.xyxy`).
- Detected class identifier (`box.cls`).
- Confidence score associated with the prediction (`box.conf`).

In this implementation, only class identifiers and bounding box coordinates are used by the decision-making logic.

### 3.2. Driving Corridor Analysis

A driving corridor is represented by a trapezoidal polygon defined based on the frame dimensions (width `L`, height `H`):

```text
(L * 0.12, H * 0.95), (L * 0.42, H * 0.55), (L * 0.58, H * 0.55), (L * 0.88, H * 0.95)
```

This zone represents an approximation of the space located in front of the vehicle. It allows distinguishing objects likely to interfere with the studied trajectory from objects detected outside this zone.

The OpenCV function `cv2.pointPolygonTest` is used to determine whether a point belongs to the polygon (return `>= 0`: point inside or on the edge).

In this approach, the bottom-center point of the bounding box `(x1 + x2) / 2, y2` is used to evaluate the object's position relative to the corridor.

### 3.3. Decision Logic (Timer-Based State Machine)

The system relies on two timers, `brake_timer` and `warning_timer`, decremented at each frame, associated with a hold duration of `COOLDOWN_FRAMES = 12` frames. Three states result:

- **CRUISING**: both timers are at zero. Normal driving state, no threat in the corridor.
- **WARNING**: `warning_timer > 0`. Activated when a sports ball is detected in the corridor **and** no braking is in progress (`brake_timer == 0`).
- **EMERGENCY_BRAKING**: `brake_timer > 0`. Activated as soon as a pedestrian is detected in the corridor.

Exact transition rules applied at each frame:

1. If a pedestrian is detected in the corridor: `brake_timer = 12` and `warning_timer = 0` (braking takes precedence over warning).
2. Otherwise, if a ball is detected in the corridor and `brake_timer == 0`: `warning_timer = 12`.
3. Otherwise: both timers are decremented by 1; when both reach zero, the system returns to the **CRUISING** state.

State priority order: pedestrian > ball > no threat.

### 3.4. Results Visualization

The processing produces, frame by frame, a visualization including:

- The driving corridor: semi-transparent cyan fill (opacity ≈ 12%) and cyan outline.
- Monitored objects located **outside** the corridor: thin green bounding box.
- Pedestrian detected **inside** the corridor: thick red bounding box and red dot on the bottom-center point.
- Ball detected **inside** the corridor: orange bounding box.
- A status bar at the bottom of the frame: red (emergency braking), orange (warning), or dark gray with green text (normal cruising).

Frames are converted from BGR (OpenCV) to RGB and then displayed directly in the notebook via Matplotlib, with cell refresh at each frame (`clear_output`).

This implementation does not display class labels, confidence scores, or numerical telemetry.

## 4. Technical Architecture

The processing pipeline is organized into several successive steps.

### Step 1: Frame Acquisition

Reading the video `source/test_driving_video.mp4` via `cv2.VideoCapture`, then extracting frame dimensions (`CAP_PROP_FRAME_WIDTH`, `CAP_PROP_FRAME_HEIGHT`).

### Step 2: Corridor Definition

A trapezoidal polygon representing the trajectory zone is calculated based on frame dimensions.

### Step 3: Object Detection

The YOLOv8n model analyzes the frame and returns available detections, including bounding boxes and classes.

### Step 4: Spatial Evaluation

For each monitored object, the bottom-center point of its bounding box is compared to the corridor using `cv2.pointPolygonTest`.

### Step 5: Decision Making

Indicators `pedestrian_in_lane` and `ball_in_lane` are set based on detections within the corridor.

### Step 6: Timer Management

Variables `brake_timer` and `warning_timer` are updated according to the transition rules defined in 3.3.

### Step 7: Results Display

OpenCV drawings (corridor, boxes, status bar) are applied to the frame, then the rendering is displayed inline in the notebook via Matplotlib.

## 5. Technologies Used

| Technology | Usage |
|---|---|
| Python | Main project language |
| Jupyter Notebook | Development and interactive execution of the pipeline |
| OpenCV | Video reading, image processing, and geometric analysis (`pointPolygonTest`, `fillPoly`, `addWeighted`) |
| NumPy | Numerical data manipulation (polygon, boxes) |
| Ultralytics YOLOv8n | Object detection |
| Matplotlib | Frame rendering in the notebook |
| IPython.display | Inline display refresh (`clear_output`) |
| State Machine (Timers) | Decision logic management |

## 6. Monitored Object Classes

The project uses the following classes from the COCO dataset supported by the YOLOv8n model.

| COCO ID | Class | Role in Implementation |
|---|---|---|
| 0 | Person | Triggers emergency braking simulation if present in the corridor |
| 32 | Sports ball | Triggers warning if present in the corridor (when no braking is active) |
| 1 | Bicycle | Monitored (displayed in green outside corridor), no decision effect |
| 3 | Motorcycle | Monitored (displayed in green outside corridor), no decision effect |
| 15 | Cat | Monitored (displayed in green outside corridor), no decision effect |
| 16 | Dog | Monitored (displayed in green outside corridor), no decision effect |
| 17 | Horse | Monitored (displayed in green outside corridor), no decision effect |
| 18 | Sheep | Monitored (displayed in green outside corridor), no decision effect |
| 21 | Bear | Monitored (displayed in green outside corridor), no decision effect |
| 22 | Zebra | Monitored (displayed in green outside corridor), no decision effect |

Only the *Person* and *Sports ball* classes trigger safety actions. The other listed classes are detected and drawn when located outside the corridor, but do not modify the system state. COCO classes not listed (cars, trucks, etc.) are ignored by the decision logic.

The COCO categories presented here constitute a selection of detectable classes and not an exhaustive definition of vulnerable road users.

## 7. Prerequisites

To run the project, the following elements are required:

- Python 3.8 or later, subject to compatibility with installed library versions.
- pip for dependency installation.
- Jupyter Notebook or JupyterLab.
- Python libraries listed in the `requirements.txt` file.
- Model weights `yolov8n.pt`: downloaded automatically by Ultralytics on first launch (Internet connection required), or to be placed manually in the working directory.
- A demonstration video available at `source/test_driving_video.mp4`, or path adaptation in the code.

Required resources must be available at the paths expected by the notebook.

## 8. Installation and Execution

### 8.1. Project Retrieval

Clone the Git repository:

```bash
git clone <REPOSITORY_URL>
```

Navigate to the project folder:

```bash
cd scenario_7_vru_detection
```

Replace `<REPOSITORY_URL>` with the actual GitHub repository address.

### 8.2. Dependency Installation

Install dependencies defined in the `requirements.txt` file:

```bash
pip install -r requirements.txt
```

If necessary, verify that Jupyter is installed:

```bash
pip install notebook
```

### 8.3. Notebook Launch

Start Jupyter Notebook:

```bash
jupyter notebook
```

In the interface that opens, select the file:

`scenario_7_vru_detection.ipynb`

JupyterLab can also be used:

```bash
jupyter lab
```

### 8.4. Pipeline Execution

Open the notebook and execute cells in the intended order.

Before execution, verify the following elements:

- Availability of `yolov8n.pt` weights (automatic download on first launch otherwise).
- Presence of the video at `source/test_driving_video.mp4` (or path adaptation in the code).
- Correct installation of dependencies.
- Python environment compatibility with the libraries used.

Processing runs until the end of the video or until manual kernel interruption (Stop button / Kernel → Interrupt). Video resources are released at the end of execution.

The exact workflow depends on the organization and content of the notebook.

## 9. Project Structure

The current repository structure is as follows:

```text
scenario_7_vru_detection/
├── assets/
├── models/
├── source/
│   └── test_driving_video.mp4
├── README.md
├── requirements.txt
└── scenario_7_vru_detection.ipynb
```

Element descriptions:

- `assets/`: folder containing project resources.
- `models/`: folder intended for model files and weights used by the system; the `yolov8n.pt` file is automatically downloaded by Ultralytics in the working directory on first launch.
- `source/`: folder containing the demonstration video used by the notebook.
- `README.md`: project documentation.
- `requirements.txt`: Python dependency list.
- `scenario_7_vru_detection.ipynb`: Jupyter notebook containing the code, processing steps, and project visualizations.

Detailed content of the `assets/`, `models/`, and `source/` folders depends on files present in the repository.

## 10. Limitations and Improvement Perspectives

### 10.1. Trajectory Estimation

The trapezoidal corridor constitutes a simplified geometric approximation, fixed by constant proportions of the frame. It does not necessarily account for camera calibration, road curvature, steering angle, or vehicle movements.

A possible improvement would be to integrate lane detection, camera calibration, and more accurate trajectory estimation.

### 10.2. Timer Management

The logic relies on a number of frames (`COOLDOWN_FRAMES = 12`) and therefore depends on the effective frame rate of the video.

Using real elapsed time would make timer management less dependent on the number of frames processed per second.

### 10.3. Spatial Analysis

The bottom-center point of a bounding box constitutes an approximation of an object's position. This method does not fully represent its shape, movement, or future trajectory.

Integration of multi-object tracking and motion estimation could improve risk analysis.

### 10.4. Collision Prediction

The system relies on simplified geometric and decision rules. A possible extension would be to estimate time-to-collision (TTC) based on distance and relative speed of objects.

This evolution would require sufficiently reliable motion and distance information.

### 10.5. Performance Evaluation

Additional tests would allow evaluating system behavior under various conditions: variable lighting, occlusions, distant objects, and complex scenes.

Envisaged metrics include precision, recall, false detection rate, missed object rate, and average frame processing time.

### 10.6. Optimization

Based on results obtained, several avenues could be explored:

- Hardware acceleration of inference.
- Video processing optimization (Matplotlib frame-by-frame rendering less performant than native video output).
- Export of annotated video in addition to inline display.
- Improvement of temporal object tracking.
- Corridor adaptation to estimated vehicle trajectory.
- Integration of risk prediction methods.

### 10.7. Safety Considerations

This project is an experimental prototype of computer vision and emergency braking simulation. It does not constitute a certified braking system and must not be used to directly command a real vehicle.

A real-world application would notably require thorough validation, safety mechanisms, reliable distance and speed estimation, and compliance with applicable automotive standards.

## 11. Conclusion

This project illustrates the integration of object detection, geometric analysis, and timer-based decision logic in a simplified autonomous driving scenario.

The combination of YOLOv8n, a driving corridor, and state management rules allows studying how visual information can contribute to initial detection of potentially dangerous situations.

This architecture provides an experimental foundation for further exploration of topics such as object tracking, collision prediction, and improvement of perception systems for autonomous driving.

## 12. Author

<div align="center">

<table>
<tr>
<td align="center">
<a href="https://github.com/mohammed-boujaada">
<img src="https://github.com/mohammed-boujaada.png" width="120px;" alt="Mohammed Boujaada"/>
<br/>
<sub><b>Mohammed Boujaada</b></sub>
</a>
<br/>
MIATE Master's — FSA Nador
<br/>
<a href="https://github.com/mohammed-boujaada">
<img src="https://img.shields.io/badge/GitHub-@mohammed--boujaada-black?logo=github"/>
</a>
<a href="https://www.linkedin.com/in/mohammed-boujaada-229a3333a/">
<img src="https://img.shields.io/badge/LinkedIn-Boujaada%20Mohammed-blue?logo=linkedin"/>
</a>
</td>
</tr>
</table>

</div>

**Institution**: FSA Nador — Master's in Artificial Intelligence & Emerging Technologies  
**Module**: Computer Vision  
**Academic Year**: 2025 - 2026

---

## License

This project is provided **as is** strictly for educational and academic purposes.  
Free to use, modify, and distribute within non-commercial, academic contexts.

---

## Acknowledgments

- **Ultralytics** — YOLOv8 framework for object detection
- **OpenCV** — Computer vision library
- **FSA Nador** — Academic supervision and foundational project requirements
- All contributors and peer reviewers