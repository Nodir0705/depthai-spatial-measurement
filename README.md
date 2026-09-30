# Box Fill Level Measurement with OAK-D

Measures how full a box is, in real time, using the stereo depth camera of a Luxonis OAK-D.

![Depth view with ROI and fill level](https://github.com/user-attachments/assets/78f29472-f6b8-4cfb-8ac7-6a7615a350a8)

## What it does

The camera looks down into a box. You draw a rectangle over the box opening, then record two reference depths: one with the box empty and one with it full. After that, the app compares the current depth with those two references and shows the fill level as a percentage and a bar.

It is based on the host-side spatial calculation example from Luxonis DepthAI. The original example reports the depth around a single point. This version averages depth over the whole selected area, which gives a steadier reading for a box surface.

## How it works

1. The two mono cameras (800p) feed the on-device `StereoDepth` node, which outputs a depth map and a disparity map.
2. On the host, `calc.py` takes the depth pixels inside the selected rectangle and drops values outside 20 cm to 30 m.
3. It computes the average, min and max depth of the remaining pixels.
4. `main.py` turns the average depth into a fill level:
   `fill % = (empty_depth - current_depth) / (empty_depth - full_depth) * 100`, clamped to 0–100.

## Quick start

Needs an OAK-D camera connected over USB and Python 3.

```bash
git clone https://github.com/Nodir0705/depthai-spatial-measurement.git
cd depthai-spatial-measurement
python3 -m pip install -r requirements.txt
python3 main.py
```

Then, in the `depth` window:

| Input | Action |
|---|---|
| Mouse drag | Draw the measurement area over the box |
| `g` | Save the current depth as the **empty** box reference |
| `f` | Save the current depth as the **full** box reference |
| `c` | Clear the area |
| `q` | Quit |

The fill level appears once both references are set.

## Project structure

```
main.py           Camera pipeline, mouse ROI selection, calibration keys, fill-level display
calc.py           Average/min/max depth inside an ROI (adapted from the Luxonis example)
utility.py        Text and rectangle drawing helpers
requirements.txt  opencv-python, numpy, depthai==2.16.0.0
```

## Notes and limitations

- Calibration is kept in memory only. It resets every time the app starts.
- Keep the camera and box in the same position after calibration.
- Lighting and surface texture affect stereo depth. Very dark or shiny contents can give invalid pixels. The overlay shows how many pixels in the area were valid.
- The fill level is linear between the two references. It assumes the contents form a roughly flat surface.
- `BoxAnalyzer(730)` sets a box height in `main.py`, but it is not used in the calculation yet.
