# Vision-Based Indoor Localisation in a Museum Environment

## Project Overview

This project explores **indoor localisation of museum visitors** using **computer vision techniques**.  
Instead of relying on GPS or indoor positioning hardware, the system determines a visitor’s location by **recognising paintings in video footage** and mapping them to known museum rooms.

The system was developed and evaluated using real-world data captured in the **Museum of Fine Arts (MSK) in Ghent**, and demonstrates that visual recognition combined with temporal modelling can achieve reliable indoor localisation.



## Objectives

- Detect paintings in video footage
- Match detected paintings with a reference database
- Infer the visitor’s current museum room
- Improve robustness using temporal context


## Frame Selection

To reduce computational cost, only **high-quality frames** are processed.

- Frame sharpness is measured using the **variance of the Laplacian**
- Frames are sampled adaptively based on video FPS
- The sharpest frame in each window is retained

### Camera Calibration

Some videos were recorded using a GoPro camera, resulting in fisheye distortion.

- Camera calibration is performed using a chessboard pattern
- Intrinsic camera parameters are estimated
- Frames are undistorted using OpenCV before further processing


## Painting Extraction

Painting extraction is performed in **two stages**.

### Rough Extraction

- A custom **Difference of Gaussians (DoG)** pipeline
- Heavy smoothing and thresholding
- Identifies large contours likely containing paintings
- Produces a zoomed region for further refinement

### Fine Extraction

- Gaussian blur, Canny edge detection, and dilation
- Contour filtering based on size constraints
- Each candidate region is scored using:
  - Colourfulness
  - Solidity (convex hull ratio)
  - Edge density

Only the highest-scoring region is retained per frame to minimise false positives.


## Painting Matching

Extracted paintings are matched against a precomputed database.

### Feature Matching

- Images are sharpened using Gaussian-based unsharp masking
- Keypoints and descriptors are extracted using:
  - AKAZE
  - ORB
  - SIFT (evaluated but discarded due to size and speed)

### Similarity Scoring

- k-NN matching with **Hamming distance**
- **Lowe’s ratio test** to remove weak matches
- Histogram comparison using **Chi-square similarity**
- Final score = keypoint score × histogram similarity

AKAZE (descriptors only) provided the best balance between accuracy and efficiency.


## Localisation with Hidden Markov Models

To stabilise predictions over time, a **Hidden Markov Model (HMM)** is applied.

- Maintains temporal consistency
- Biases predictions toward:
  - The same room
  - Adjacent rooms
  - Logical visitor paths
- Helps recover from occasional matching errors

This significantly improves localisation stability in continuous video sequences.


## Results

### Matching Performance

| Method | Database Size | Correct Matches |
|------|---------------|----------------|
| AKAZE (descriptors) | ~560 MB | 23 / 27 |
| ORB (descriptors) | ~45 MB | 17 / 27 |

### Localisation Accuracy

- Overall accuracy of approximately **76%**
- Comparable performance on smartphone and GoPro footage
- Temporal modelling proved essential for correcting misclassifications

## Key Takeaways

- Computer vision is a viable alternative for indoor localisation
- Two-stage painting extraction improves robustness
- AKAZE offers an effective balance between precision and speed
- Temporal modelling is crucial for real-world deployment


## Team

- Cédric Bruylandt  
- Bart Ertveldt  
- Arthur Leclercq  
- Senne Loobuyck  
- Ziggy Verhulst 
