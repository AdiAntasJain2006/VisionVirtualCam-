# VideoBroadcaster

VideoBroadcaster uses YOLOv8 instance segmentation to isolate people from a live camera feed and send the processed video to a virtual camera. Use that virtual camera as your camera source in video-conferencing or recording applications.

## Features

- Select an available camera.
- Segment people in the camera feed.
- Choose a black background, a blurred background, or the bundled office background.
- Send the processed frames to a virtual camera with `pyvirtualcam`.
- Control the stream from a local browser interface.

## Requirements

- Python 3.11 (the setup below uses Conda).
- A webcam or other OpenCV-compatible camera.
- A PyTorch installation compatible with your operating system and hardware.
- A virtual-camera provider supported by `pyvirtualcam`. On Windows, for example, install OBS Studio and start **OBS Virtual Camera** before using the output in another application.
- The YOLO segmentation checkpoint `yolov8m-seg.pt` in the project root. `engine.py` loads this file by name.

## Setup

Run these commands from the project directory.

1. Create and activate an environment:

   ```bash
   conda create -n broadcaster python=3.11
   conda activate broadcaster
   ```

2. Install PyTorch and TorchVision for your machine. For an NVIDIA GPU with CUDA 12.4, the project’s original setup uses:

   ```bash
   conda install pytorch==2.5.1 torchvision==0.20.1 pytorch-cuda=12.4 -c pytorch -c nvidia
   ```

   For CPU-only use, install the CPU-compatible PyTorch and TorchVision packages for your platform instead. See the [official PyTorch installation selector](https://pytorch.org/get-started/locally/) for the appropriate command.

3. Install the remaining Python packages:

   ```bash
   pip install -r requirements.txt
   ```

4. Make sure `yolov8m-seg.pt` is available in the project root alongside `engine.py`. The application does not download the checkpoint automatically.

## Run

From the project directory, start the FastAPI server:

```bash
python main.py
```

Open [http://localhost:8000](http://localhost:8000) in your browser. In the controller:

1. Select **List Devices** and choose a camera.
2. Choose the frame rate and background mode.
3. Select **Start Stream**.
4. Select the virtual camera in your meeting or recording application.
5. Select **Stop Stream** when finished.

The server listens on `0.0.0.0:8000`. Keep it running while using the virtual camera. Camera access and virtual-camera availability depend on the operating system and installed drivers.

## HTTP endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/` | Serve the browser controller. |
| `GET` | `/devices` | List detected camera devices (the app checks camera indices 0 through 4). |
| `GET` | `/start` | Start processing a camera feed. |
| `GET` | `/stop` | Stop the stream. |

`/start` accepts these query parameters:

| Parameter | Default | Description |
| --- | --- | --- |
| `source` | `0` | Camera index to open. |
| `fps` | `15` | Requested output frame rate (1–60). |
| `blur_strength` | `21` | Gaussian blur kernel strength when using the blur background. |
| `background` | `none` | Background mode: `none` (black), `blur`, or `default` (bundled office image). |

For example:

```text
http://localhost:8000/start?source=0&fps=15&blur_strength=21&background=blur
```

The selected camera's available frame rate may limit the actual output rate.

## Project files

- `main.py` — FastAPI application and HTTP routes.
- `stream_utils.py` — camera capture, frame processing loop, and virtual-camera output.
- `engine.py` — YOLOv8 person segmentation and background effects.
- `static/index.html` — browser controller.
- `static/default-office-animated.png` — bundled replacement background.
- `requirements.txt` — Python package dependencies (PyTorch must be installed separately).
