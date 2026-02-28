# 🏃 Sports Player Tracking & Analytics using YOLOv8

## What is this project?

Ever watched a football match and wondered how analysts track every player's movement across the pitch? This project does exactly that — using just a video file and a deep learning model.

It detects every player on screen, assigns them a unique ID, and draws their movement trail in real time as the video plays. The whole thing runs inside a Jupyter Notebook, so you don't need any special setup beyond Python and a few libraries.

---

## What can it do?

Here's what the system handles out of the box:

- Detects all players in every frame using YOLOv8
- Assigns each player a persistent tracking ID across frames
- Draws movement trails (blue lines) showing where each player has been
- Displays a live player count on the video
- Shows the annotated video frame-by-frame inside Jupyter Notebook
- Works with any sports footage — football, basketball, rugby, you name it

---

## How does it work?

### Detecting and tracking players

YOLOv8 processes each frame and looks specifically for people (COCO class `0`). Every detected person gets a bounding box and a persistent ID — so if Player 3 walks off screen and comes back, the system recognizes them as the same person.

```python
PERSON_CLASS = 0   # COCO class ID for "person"
```

Only people are tracked — the ball, goalposts, and other objects are deliberately ignored to keep things clean.

---

### Drawing movement trails

Every time a player moves, the system records the center point of their bounding box. Over time, these points build up into a trail that shows exactly where they've been on the field.

```
Center point = ((x1 + x2) / 2,  (y1 + y2) / 2)
```

The trail is drawn as a series of connected blue lines, making it easy to see patterns like sprints, positioning shifts, and covering distances at a glance.

---

### What gets shown on each frame

- 🟩 A green bounding box around each detected player
- `Player {ID}` label above every box
- 🔵 A blue movement trail following each player's path
- A live **Players:** count in the top-left corner

---

## Project structure

```
sports-analytics/
│
├── sports_analytics.ipynb   ← the main notebook
├── Sports.mp4               ← your input sports video
├── yolov8n.pt               ← YOLOv8 model weights
└── README.md
```

---

## Getting started

### Step 1 — Install the dependencies

```bash
pip install ultralytics opencv-python numpy matplotlib
```

### Step 2 — Drop in your files

Place your sports video and model weights in the same folder as the notebook:

```
Sports.mp4    ← any sports footage works (football, basketball, etc.)
yolov8n.pt    ← Ultralytics will auto-download this if it's missing
```

### Step 3 — Run the notebook

Open the notebook in Jupyter Notebook or JupyterLab and run the single cell. Frames will render inline as the video processes — you'll see player boxes and trails updating in real time.

### Step 4 — Watch it go

Once it's done, the notebook will print:

```
Sports analytics finished!
```

---

## Tweaking the settings

The notebook is kept intentionally simple — just change these two lines at the top to point to your own files:

```python
model = YOLO("yolov8n.pt")       # swap for yolov8s.pt or yolov8m.pt for better accuracy
video_path = "Sports.mp4"        # point this to your own video file
```

And if you want to adjust detection sensitivity:

```python
results = model.track(frame, persist=True, conf=0.3)
# conf=0.3 means 30% confidence threshold — raise it to reduce false detections
```

---

## Libraries used

| Library            | Version |
|--------------------|---------|
| Python             | 3.13+   |
| OpenCV             | 4.x+    |
| Ultralytics YOLOv8 | 8.x+    |
| NumPy              | 2.x+    |
| Matplotlib         | 3.x+    |

---

## A few things worth knowing

- The model tracks **people only** — it ignores everything else in the frame by design.
- Movement trails grow longer as the video plays. For very long videos, you may want to limit trail length to the last N points to keep it readable.
- Switching from `yolov8n.pt` (nano) to `yolov8s.pt` (small) or `yolov8m.pt` (medium) gives noticeably better player detection, especially in crowded scenes — at the cost of a bit more processing time.
- No output video is saved in this version — it displays inside the notebook only. If you want to save the result, you can add a `cv2.VideoWriter` block similar to the traffic project.
