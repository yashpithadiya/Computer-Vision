# 🚦 Traffic Monitoring

## What is this project?

Ever sat in traffic and wondered if there was a smarter way to detect and respond to congestion? That's exactly what this project tries to solve — using a camera feed and a bit of deep learning.

This notebook analyzes traffic footage in real time. It spots vehicles, follows them across frames, estimates how fast they're moving, and figures out whether the road is flowing freely or heading toward a gridlock. The whole thing runs inside a Jupyter Notebook and spits out an annotated video you can review afterward.

---

## What can it do?

Here's a quick rundown of what's happening under the hood:

- Detects cars, motorcycles, buses, and trucks using YOLOv8
- Assigns each vehicle a unique ID and tracks it across frames
- Estimates speed by measuring how far each vehicle moves between frames
- Smooths out jittery speed readings using a short moving average
- Automatically forgets vehicles that disappear from the scene
- Raises a congestion alert when too many vehicles are moving slowly for too long
- Labels traffic as **FREE FLOW**, **MODERATE**, or **HEAVY** based on vehicle count
- Shows everything live in the notebook and saves the result as `output.mp4`

---

## How does it work?

### Detecting and tracking vehicles

YOLOv8 scans each frame and draws boxes around vehicles. It also assigns persistent IDs, so the system knows it's looking at the same car from one frame to the next — not a brand new one every time.

The vehicle types it looks for (from the COCO dataset):

| Class ID | Vehicle Type |
|----------|-------------|
| 2        | Car         |
| 3        | Motorcycle  |
| 5        | Bus         |
| 7        | Truck       |

---

### Estimating speed

Speed is calculated by looking at how far the center of a vehicle's bounding box moves between two consecutive frames:

```
distance = sqrt((cx - px)² + (cy - py)²)
speed    = distance × FPS
```

To avoid wild fluctuations from a single noisy frame, the last 5 speed readings are averaged together. It's not GPS-accurate, but it gives a solid sense of whether traffic is crawling or moving.

> **Note:** Speed is in pixels per second, not km/h. Converting to real-world units would require knowing the camera's position and angle.

---

### Deciding if there's congestion

The system watches for a combination of two things: a lot of vehicles on screen, and most of them barely moving. If that situation holds for enough consecutive frames, it declares congestion.

| Parameter           | Default |
|---------------------|---------|
| Low speed threshold | 25 px/s |
| Min vehicles needed | 8       |
| Min slow vehicles   | 5       |
| Frames to confirm   | 40      |

Traffic level is also displayed based purely on vehicle count:

| Vehicles on screen | Traffic Level |
|--------------------|---------------|
| Fewer than 5       | FREE FLOW     |
| 5 to 11            | MODERATE      |
| 12 or more         | HEAVY         |

---

## Project structure

```
traffic-congestion-detection/
│
├── traffic_detection.ipynb   ← the main notebook
├── video.mp4                 ← your input traffic video
├── yolov8n.pt                ← YOLOv8 model weights
├── output.mp4                ← annotated output (created after running)
└── README.md
```

---

## Getting started

### Step 1 — Install the dependencies

```bash
pip install ultralytics opencv-python numpy matplotlib
```

### Step 2 — Drop in your files

Put your traffic video and the model weights in the same folder as the notebook:

```
video.mp4     ← any traffic footage works
yolov8n.pt    ← Ultralytics will auto-download this if it's missing
```

### Step 3 — Run the notebook

Open `traffic_detection.ipynb` in Jupyter Notebook or JupyterLab and run the cells top to bottom. Each frame will display inline as it's processed.

### Step 4 — Check the output

When it's done, you'll find the annotated video saved as `output.mp4` in the same directory.

---

## Tweaking the settings

All the key parameters are at the top of the notebook, so they're easy to find and adjust:

```python
VIDEO_PATH = "video.mp4"          # swap in your own video path
MODEL_PATH = "yolov8n.pt"         # use yolov8s.pt or yolov8m.pt for more accuracy

VEHICLE_CLASSES = [2, 3, 5, 7]    # car, motorcycle, bus, truck

LOW_SPEED_THRESHOLD = 25          # below this speed, a vehicle counts as "slow"
MIN_VEHICLES = 8                  # minimum vehicles on screen to consider congestion
MIN_LOW_SPEED = 5                 # minimum slow vehicles to consider congestion
CONGESTION_FRAMES = 40            # how many frames in a row before alerting
```

---

## What you'll see on screen

Each processed frame shows:

- 🟩 A green box drawn around every detected vehicle
- The vehicle's tracking ID just above the box
- Its current estimated speed just below the box
- A live count of total vehicles and slow-moving vehicles
- The current traffic level label in the corner
- A bold red **CONGESTION DETECTED** warning when thresholds are met

---

## Libraries used

| Library            | Version |
|--------------------|---------|
| Python             | 3.13+   |
| OpenCV             | 4.13+   |
| Ultralytics YOLOv8 | 8.3+    |
| PyTorch            | 2.9+    |
| NumPy              | 2.1+    |
| Matplotlib         | 3.10+   |

---

## A few things worth knowing

- If you want better detection accuracy at the cost of some speed, swap `yolov8n.pt` for `yolov8s.pt` or `yolov8m.pt`.
- The model only tracks the four vehicle types listed above — pedestrians, cyclists, and other objects are intentionally ignored.
- If a vehicle disappears for more than 60 frames, the system cleans up its data to keep memory usage tidy.
