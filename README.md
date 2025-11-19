# Computer-Vision-Multi-Face-Blink-Yawn-Counter-using-Color-Coded-Media-Pipe-Mesh.

This project is a **real-time multi-face analysis tool** built using **OpenCV**, **MediaPipe Face Mesh**, **NumPy**, and **SciPy**.

It detects:

- **Eye blinks (per eye, per face)** using the **Eye Aspect Ratio (EAR)**
- **Mouth openings** using a simple **lip distance metric (MAR-like)**
- Tracks **multiple faces simultaneously** and overlays **live counters & metrics** on the video feed.

The app runs on your **webcam** and displays an annotated preview window with all metrics.

---

## 🔍 Features

- ✅ Real-time **multi-face detection** using MediaPipe Face Mesh  
- ✅ **Per-face** tracking using index-based `face_id`  
- ✅ Eye blink detection using **EAR**:
  - Separate counters for **left** and **right** eye  
- ✅ Mouth-open detection using lip distance (`MAR_THRESHOLD`)  
- ✅ On-screen overlays:
  - Face ID  
  - Average EAR  
  - Left / Right blink counts  
  - Mouth open count + current lip distance  
  - Current FPS  
- ✅ Visual overlays for:
  - Eye landmarks
  - Lip landmarks
  - Face mesh tesselation

---

## 🧠 How It Works (High-Level)

1. **Face Mesh Detection**
   - Uses `mp.solutions.face_mesh.FaceMesh` with:
     - `max_num_faces=10`
     - `refine_landmarks=True`
     - Good detection & tracking confidence (`0.7`)

2. **Eye Aspect Ratio (EAR)**
   - Uses 6 landmark points per eye:
     - **Right eye indices:** `Right_Eye = [33, 160, 158, 133, 153, 144]`  
     - **Left eye indices:** `Left_Eye  = [362, 385, 387, 263, 373, 380]`
   - EAR Formula:
     \[
     EAR = \frac{‖p_2 - p_6‖ + ‖p_3 - p_5‖}{2 \cdot ‖p_1 - p_4‖}
     \]
   - When EAR drops below `EAR_THRESHOLD` → considered a blink.

3. **Mouth Opening (Lip Distance)**
   - Uses two lip points:
     - `Lip_Points = [13, 14]` → upper & lower inner lip
   - Computes **Euclidean distance** between these two points:
     ```python
     normalized_lip_distance(u, l)
     ```
   - When distance > `MAR_THRESHOLD` → mouth is considered "open".

4. **Per-Face Counters**
   - `counters` dict stores blink/mouth info **for each `face_id`**:
     ```python
     counters[face_id] = {
         "L": 0, "R": 0, "M": 0,        # counts
         "prev_L": 1, "prev_R": 1,      # eye state in previous frame
         "prev_M": 0                    # mouth state in previous frame
     }
     ```
   - Logic avoids double-counting:
     - Blinks are counted when going from **open → closed**.
     - Mouth opens are counted when going from **closed → open**.

5. **FPS Calculation**
   - Uses `time.time()` to measure frame processing time and compute:
     ```python
     fps = 1 / (c_time - p_time)
     ```
   - FPS is shown live on the video frame.

---

## 🧩 Project Structure

The main script contains:

- **Configuration constants**
  - `EAR_THRESHOLD`
  - `MAR_THRESHOLD`
  - Eye & lip landmark indices
  - Text drawing properties

- **Helper functions**
  - `ear(pts)` – computes Eye Aspect Ratio.
  - `normalized_lip_distance(u, l)` – computes lip distance between 2 landmarks.
  - `pixel_landmarks(face, w, h)` – converts normalized landmarks to pixel coords.
  - `face_box(face, w, h)` – computes bounding box for each face.

- **Core function**
  - `start_detection()` – handles webcam, detection loop, drawing, and counters.

- **Entry point**
  - At the end:
    ```python
    if __name__ == "__main__":
        start_detection()
    ```

---

## 🛠 Requirements

Make sure you have:

- **Python** ≥ 3.8
- The following Python packages:

```bash
pip install opencv-python mediapipe numpy scipy
