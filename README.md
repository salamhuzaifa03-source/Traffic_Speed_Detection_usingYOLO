# Traffic_Speed_Detection_usingYOLO
Python-based vehicle speed screening prototype using YOLO, ByteTrack, OpenCV, and perspective mapping on roadside traffic video.


Incoming Vehicle Speed Review - Run Instructions -Muhammad Shah 
============================================================

What this package does
----------------------
This script processes a traffic video, tracks incoming vehicles in the calibrated road patch,
estimates a screening speed value, writes an annotated output video, and saves a plain-text log.

Files you should have in the same project folder
------------------------------------------------
1. Your Python script (the speed review script)
2. traffic.mp4
3. yolov8s.pt   (or let Ultralytics download it automatically)
4. output/      (this will be created automatically if missing)

Required Python packages
------------------------
Install these first:

pip install ultralytics opencv-python numpy

If you want GPU acceleration, make sure your PyTorch / Ultralytics install is using a CUDA-enabled build.

Before you run the script
-------------------------
Check these values near the top of the code:

- SOURCE_VIDEO = "traffic.mp4"
  Change this if your video file has a different name.

- YOLO_WEIGHTS = "yolov8s.pt"
  Change this if you are using a different YOLO weights file.

- SCENE_START_CLOCK = "11:25:00 PM"
  This is the time used for the text log. Change it to match the real time you want frame 0 to represent.

- INCOMING_FLOW = "down"
  Use "down" if vehicles move from top to bottom in the frame.
  Use "up" if vehicles move from bottom to top.

- LANE_PATCH_REL = np.float32([...])
  These are the normalized ROI points from calibration.
  They define the active road patch used for speed review.

- ROAD_PATCH_WIDTH_M and ROAD_PATCH_LENGTH_M
  These are the real-world dimensions assumed for the calibrated road patch.
  Better measurements here will improve speed estimates.

- SHOW_LIVE_VIEW = True
  Set this to False if you do not want the live OpenCV window.

How to run
----------
Open a terminal or PowerShell window in the project folder and run:

python your_script_name.py

Example:
python incoming_speed_review.py

While it is running
-------------------
- A live window will open if SHOW_LIVE_VIEW is True.
- Press q to stop the run early.
- The script will keep tracking vehicles, drawing the ROI, and overlaying the speed estimate.

What gets saved
---------------
The script creates these output files:

1. output/incoming_speed_annotated.mp4
   This is the processed video with:
   - tracked vehicles
   - IDs
   - ROI overlay
   - screening speed text

2. output/vehicle_log.txt
   This is the review log with:
   - track ID
   - vehicle class
   - first seen time
   - last seen time
   - duration on screen
   - peak screening speed when available

How the speed estimate works
----------------------------
The script does not treat the raw camera as a legal speed sensor.
Instead, it:

1. Detects and tracks vehicles with YOLO + ByteTrack
2. Uses the bottom-center of each box as the road-contact point
3. Projects that point into a bird's-eye road plane
4. Measures recent along-road displacement
5. Converts distance / time into a screening speed estimate
6. Smooths the result with a short median window

Important limitations
---------------------
- This is a screening / review prototype, not certified enforcement hardware.
- Accuracy depends on the road patch calibration and real-world dimensions.
- Night glare, vibration, occlusion, and distant vehicles can affect stability.
- For stronger enforcement logic, the next step is multi-camera average-speed verification.

Quick troubleshooting
---------------------
Problem: "Could not open video"
Fix:
- Make sure traffic.mp4 exists
- Check the file path
- Confirm the file is not open in another application

Problem: output video not created
Fix:
- Confirm the script finished properly
- Make sure the output folder can be written to
- Check that VideoWriter opened successfully

Problem: speed values look too high or too low
Fix:
- Re-check ROAD_PATCH_WIDTH_M and ROAD_PATCH_LENGTH_M
- Re-check the LANE_PATCH_REL calibration points
- Make sure the road patch only covers the usable lane area

Problem: too many false detections
Fix:
- Raise DETECT_CONF slightly
- Reduce the ROI to the cleanest lane area
- Use a cleaner or brighter source video if available

Recommended folder layout
-------------------------
project_folder/
    incoming_speed_review.py
    traffic.mp4
    output/
        incoming_speed_annotated.mp4
        vehicle_log.txt

End of instructions
