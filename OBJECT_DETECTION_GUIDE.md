# Ultrasound Object Detection Feature Guide

## Overview

The Object Detection feature provides computer vision algorithms for detecting geometric objects in ultrasound images, including lines (water bath boundaries, needles), circles (calibration spheres), and horizontal interfaces (tissue layers). The feature is fully integrated into the application via a dedicated tab with interactive controls.

## Detection Algorithms

### Line Detection

**Hough Line Detection**
- Fast probabilistic transform-based detection
- Detects multiple straight lines simultaneously
- Best for clean images where speed is critical
- Suitable for needle tracking and boundary detection

**RANSAC Line Detection**
- Iterative robust detection method
- Detects up to 20 lines ordered by prominence (most inliers first)
- Handles all angles: horizontal, vertical, and diagonal
- Automatically filters noise and outliers
- Slower than Hough but significantly more accurate in noisy ultrasound images
- First detected line typically represents the most prominent feature (e.g., water bath bottom)
- Uses perpendicular distance calculation with 3-pixel inlier threshold
- Prevents division-by-zero errors through angle-aware line extension

**Horizontal Interface Detection**
- Specialized algorithm using histogram projection
- Detects multiple horizontal features by analyzing edge pixel distribution per row
- Very fast, optimized for horizontal boundaries only
- Ideal for tissue layers and water bath interfaces

### Circle/Sphere Detection

**Hough Circle Detection**
- Accurate detection for perfect circular shapes
- Voting-based algorithm for centers and radii
- Slower but precise for calibration sphere detection
- Includes Gaussian blur preprocessing to reduce noise

**Blob Sphere Detection**
- Contour-based detection with circularity analysis
- Works with irregular or imperfect circular shapes
- Very fast compared to Hough circles
- Filters by area (100-10,000 pixels) and circularity (>0.6)
- Provides circularity scores for quality assessment

## Image Preprocessing

All algorithms apply enhanced preprocessing:
- Automatic grayscale conversion
- Histogram equalization for contrast enhancement (critical for ultrasound images with varying brightness)
- Bilateral filtering to reduce noise while preserving edges

## User Interface

### Layout
- **Left Panel:** Displays original image and detection results side-by-side
- **Right Panel:** Algorithm selection, adjustable parameters, and results text output

### Controls
- **Load Current Frame:** Captures frame from active video stream
- **Load Test Frame:** Loads embedded test image (needle.png) for demonstration
- **Algorithm Dropdown:** Select from five detection methods
- **Run Detection:** Executes selected algorithm
- **Show Visualization:** Toggle annotated output with detected objects

### Adjustable Parameters

**Edge Detection (Canny)**
- Lower Threshold (10-200): Controls edge sensitivity
- Upper Threshold (50-300): Defines strong edges

**Line Detection (Hough)**
- Hough Threshold (20-200): Minimum votes required for line detection

**Circle Detection**
- Min Radius (5-100 pixels): Smallest circle to detect
- Max Radius (50-400 pixels): Largest circle to detect

## Integration Architecture

### Controller Connections
- `ObjectDetectionController` connects to `VideoController` via `ImageDataManager`
- Enables real-time frame capture from video streams
- Injection happens in `MainController.initialize()`

### Tab Integration
- Object Detection tab pre-loaded in `MainView.fxml`
- Menu item in Window > Show View > Object Detection
- Tab selection handled through `MainController.openObjectDetectionView()`

### Processing Flow
1. User loads video in Video Input tab
2. Switches to Object Detection tab
3. Loads current frame (freezes frame via Mat.clone())
4. Selects algorithm and adjusts parameters via sliders
5. Runs detection in background thread (prevents UI freezing)
6. Views results in text area and annotated visualization

## Detection Results

### Line Detection Output
Each detected line includes:
- Start and end coordinates
- Angle in degrees
- Length in pixels
- (RANSAC only) Inlier count and confidence percentage

### Circle Detection Output
Each detected circle includes:
- Center coordinates (x, y)
- Radius in pixels
- (Blob only) Circularity score (1.0 = perfect circle)

### Visualization
- Lines drawn in different colors (Red, Green, Blue, Yellow, Magenta, Cyan)
- Circles shown with outline, center point, and radius label
- RANSAC lines labeled with line number and angle
- All visualizations render on cloned image to preserve original

## Use Cases

**Water Bath Bottom Detection**
- Use RANSAC Line Detection with lower Canny thresholds (30, 100)
- First detected line represents the most prominent horizontal feature
- Extract average Y-coordinate for water level

**Needle Tracking**
- Use Hough Line Detection for speed or RANSAC for accuracy
- Set minimum line length to 100 pixels, max gap to 15 pixels
- Filter results by length to find longest line (needle shaft)
- Calculate angle and position from endpoints

**Calibration Sphere Detection**
- Use Hough Circle Detection for perfect spheres
- Set appropriate radius range based on sphere size
- Results sorted by size for multi-sphere calibration
- Increase param2 (accumulator threshold) to reduce false positives

**Multi-Object Scenarios**
- RANSAC iteratively detects multiple lines by removing inliers after each detection
- Hough methods detect all objects simultaneously
- Results can be filtered by position, size, or angle

## Performance Characteristics

**Speed Comparison**
- Fastest: Blob Detection, Horizontal Interfaces, Hough Lines (10-40ms)
- Medium: RANSAC Line Detection (20-50ms per line)
- Slowest: Hough Circle Detection (50-150ms)

**Accuracy in Noisy Images**
- Highest: RANSAC (excellent outlier rejection)
- Good: Hough methods with proper thresholding
- Variable: Blob detection (depends on contrast)

## Parameter Tuning Guidelines

**Too Many False Detections**
- Increase Canny thresholds (move sliders right)
- Increase Hough threshold for more selective detection
- Increase circle param2 (accumulator threshold)
- Add post-processing filters by size, angle, or position

**Missing Real Objects**
- Decrease Canny thresholds (move sliders left)
- Decrease Hough threshold for more sensitive detection
- Decrease circle param2
- Try alternative algorithm (e.g., Blob instead of Hough)

**Unstable Frame-to-Frame Results**
- Increase detection thresholds for consistency
- Use RANSAC instead of Hough for better stability
- Process every Nth frame instead of every frame
- Implement temporal filtering or tracking

## Implementation Files

**Core Algorithm**
- `UltrasoundObjectDetector.java`: Main detector class with all algorithms
- `UltrasoundDetectionExample.java`: Practical usage examples for common tasks

**UI Components**
- `ObjectDetectionController.java`: JavaFX controller handling UI logic and background processing
- `ObjectDetectionView.fxml`: UI layout with split-panel design
- `MainView.fxml`: Integration point for Object Detection tab
- `MainController.java`: Tab switching and controller injection

**Supporting Code**
- `ImageDataProcessor.java`: Null safety checks added to prevent crashes
- `VideoController.java`: Exposes `getDataManager()` for frame access
- `needle.png`: Embedded test image resource

**Testing**
- `UltrasoundObjectDetectorTest.java`: Unit tests for detector initialization

