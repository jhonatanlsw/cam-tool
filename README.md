# cam-tool

Tool for automated measurement of liquid droplet dimensions
in laboratory images and videos.

Conceptually based on ContactAngleMeasurement by Mike Phillips
(https://github.com/MikePhillips123/ContactAngleMeasurement),
rewritten from scratch with its own architecture in Python 3.

Licensed under GPLv3.

## Description

cam-tool analyzes videos of droplets deposited on solid
surfaces and calculates, for each selected frame:

· Droplet width (base diameter)
· Droplet height
· Radius of the sphere containing the cap (R)
· Cap volume
· Contact area with the surface
· Contact angle (spherical cap formula)

The physical model assumed is the spherical cap approximation
(sessile drop), suitable for small droplets where gravity is
negligible.

## Features

· Automatic droplet segmentation via Otsu thresholding
· Contour detection via OpenCV
· Robust baseline fitting with RANSAC
· Measurement of width, height, radius, volume, and area
· Contact angle calculation via spherical cap formula
· Scale calibration (nanometers per pixel)
· Graphical interface with Dear PyGui
· On-demand preview of the analyzed image
· Export of results to CSV and XLSX
· Slideshow generation in GIF or MP4
· Persistent settings in Settings.txt

## Requirements

· Python 3.10 or higher
· Ubuntu 22.04 or higher (tested on 26.04)
· System libraries:
  · python3-venv
  · python3-full
  · python3-tk
  · libmediainfo0v5
· Python libraries (installed via pip):
  · dearpygui
  · opencv-python
  · numpy
  · pillow
  · pymediainfo
  · pandas
  · openpyxl

## Installation

Clone the repository:

```bash
git clone https://github.com/lsjhonatan/cam-tool.git
cd cam-tool
```

Run the installation script:

```bash
./dependencias.sh
```

The script will:

1. Install system dependencies via apt
2. Create the virtual environment in .venv
3. Install Python dependencies via pip
4. Add aliases to ~/.bashrc

After installation, reload the shell:

```bash
source ~/.bashrc
```

## Usage

Activate the virtual environment and run:

```bash
cam-tool
```

Or directly:

```bash
cd ~/cam-tool
source .venv/bin/activate
python3 -m cam_tool
```

## Workflow

1. Select the video file using the "Select video" button
2. Set the scale in nanometers per pixel (camera calibration)
3. Adjust the region of interest (ROI) in the x1 and x2 fields
4. Click "Update Preview" to view the analyzed frame
5. Adjust the analysis thresholds if necessary
6. Configure the number of images, interval, and output format
7. Click "Compile Slideshow" to process all frames

## Generated outputs

Results are saved in ~/cam-tool/Output/<video_name>/:

· Images/ — annotated images of each frame (PNG)
· <video_name>.gif or <video_name>.mp4 — slideshow
· <video_name>_Medidas_[timestamp].xlsx — spreadsheet with the measurements

## Project structure

```
cam-tool/
├── README.md
├── LICENSE
├── requirements.txt
├── dependencias.sh
├── Settings.txt
├── cam_tool/
│   ├── __init__.py
│   ├── __main__.py
│   ├── config.py
│   ├── log.py
│   ├── image.py
│   ├── video.py
│   ├── segmentation.py
│   ├── contour.py
│   ├── baseline.py
│   ├── measurements.py
│   ├── overlay.py
│   ├── pipeline.py
│   ├── export.py
│   ├── slideshow.py
│   └── gui/
│       ├── __init__.py
│       ├── app.py
│       ├── settings_tab.py
│       ├── logging_tab.py
│       ├── preview.py
│       └── widgets.py
├── tests/
└── examples/
```

## Architecture

The project is organized into modules with well-defined responsibilities.

Infrastructure modules

· config.py: persistent parameter management
· log.py: unified logging with callback support (GUI)

### Input modules

· image.py: image reading and writing
· video.py: video reading and frame extraction

### Processing modules

· segmentation.py: droplet segmentation by threshold
· contour.py: extraction of the largest contour
· baseline.py: baseline fitting with RANSAC
· measurements.py: calculation of measurements (width, height, radius,
  volume, area, angle)

### Output modules

· overlay.py: drawing of visual elements on the image
· export.py: export to CSV and XLSX
· slideshow.py: GIF and MP4 assembly

### Orchestration

· pipeline.py: DropletAnalyzer class that orchestrates the complete
  analysis pipeline

### Interface

· gui/: graphical interface with Dear PyGui

### Methodology

The analysis pipeline follows these steps:

1. Acquisition: the frame is loaded and rotation correction is
   applied based on the video metadata.
2. Segmentation: the image is converted to grayscale,
   filtered with Gaussian blur, inverted, and binarized using the
   Otsu method. Morphological opening and closing operations
   remove noise and fill holes. The region of interest is
   applied as a mask.
3. Contour extraction: the largest contour is extracted with
   cv2.findContours and filtered by the region of
   interest boundaries.
4. Baseline fitting: the lowest N% of the contour are
   selected and a line is fitted with RANSAC (Random Sample
   Consensus), which is robust to outliers.
5. Measurement:
   · Width: horizontal distance between the two contact points
     (contour-baseline intersection)
   · Height: vertical distance between the top of the droplet and
     the baseline
   · Radius: R = (a² + h²) / (2h), where a = width/2 and h = height
   · Volume: V = π h² (3R - h) / 3
   · Area: A = π a²
   · Angle: θ = 2 · atan(h / a)
6. Rendering: overlay of visual elements (contour,
   baseline, dimension lines, measurement labels).
7. Export: generation of annotated images, slideshow, and
   results spreadsheet.

### Calibration

The conversion from pixels to nanometers is done using a factor
provided by the user. The value must be obtained by calibrating
the camera with a reference object of known dimensions.

The "Scale (nm/px)" field in the interface defines the factor. For example,
if 1 pixel equals 500 nanometers, the value should be 500.

### License

This project is distributed under the GNU General Public License v3.0.
See the LICENSE file for the full text.