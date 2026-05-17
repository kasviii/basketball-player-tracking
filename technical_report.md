# Technical Report — Multi-Object Detection and Persistent ID Tracking in Basketball Footage

**Assignment:** AI / Computer Vision / Data Science  
**Candidate:** Kasvi Rajwar  
**Date:** May 2026  
**Video Source:** Pexels free stock footage (outdoor night basketball, 17 seconds, 4K UHD, 424 frames)

---

## 1. Model and Detector Used

**Detector: YOLOv8n (You Only Look Once — version 8, nano variant)**

YOLOv8 by Ultralytics is a state-of-the-art single-stage object detector. The nano (`n`) variant was chosen for its speed and efficiency, making it well-suited for processing video frame by frame on a GPU-accelerated environment (Google Colab T4).

Detection was restricted to class 0 (person) only, filtering out all non-human detections. A confidence threshold of 0.30 and IoU threshold of 0.50 were applied to balance sensitivity and precision in a low-light, crowded scene.

**Tracking Algorithm: ByteTrack**

ByteTrack is a multi-object tracking algorithm that associates detections across frames using both high-confidence and low-confidence detections. Unlike trackers that discard low-confidence boxes entirely, ByteTrack uses them to maintain track continuity during partial occlusion or brief disappearances. It is integrated natively into the Ultralytics YOLOv8 pipeline via the `bytetrack.yaml` configuration.

---

## 2. Why This Combination Was Selected

YOLOv8 + ByteTrack was chosen for three reasons:

**Simplicity:** Both are available within the Ultralytics library, requiring no separate installation or complex integration. A single `model.track()` call handles detection and tracking together.

**Speed:** YOLOv8n processes frames at approximately 11ms per frame on a T4 GPU (~90 FPS), making it practical for real-time or near-real-time applications.

**Robustness:** ByteTrack is specifically designed to handle real-world tracking challenges such as occlusion and temporary disappearances, which are frequent in sports footage where players constantly move in front of each other.

---

## 3. How ID Consistency Is Maintained

ByteTrack maintains ID consistency through a **Kalman Filter** combined with **Hungarian Algorithm** matching:

- A Kalman Filter predicts where each tracked player will be in the next frame based on their current velocity and position
- The Hungarian Algorithm then optimally matches new detections to existing tracks using IoU (Intersection over Union) between predicted and actual bounding boxes
- Tracks that temporarily lose a detection (due to occlusion) are kept alive for a buffer period before being deleted
- Low-confidence detections that would normally be discarded are used as secondary matching candidates, helping recover lost tracks

In the processed video, players consistently retain their assigned IDs across frames as long as they remain visible and unoccluded.

---

## 4. Results

| Metric | Value |
|---|---|
| Total frames processed | 424 |
| Unique IDs assigned | 43 |
| Average detections per frame | 8.2 |
| Peak detections in one frame | 12 |
| Average inference time | ~11ms/frame |

The 43 unique IDs across approximately 10 actual players reflects the core challenge of persistent tracking: when a player exits the frame and re-enters, ByteTrack assigns a new ID since the track was lost. This is a known limitation of detection-based trackers without a dedicated ReID (re-identification) module.

---

## 5. Challenges Faced

**Low-light conditions:** The video was filmed at night under floodlights. Players further from the light source had lower detection confidence (as low as 0.36), occasionally causing missed detections and track loss.

**Player occlusion:** Basketball involves frequent close contact and overlapping players. When two players overlap significantly, the detector sometimes merges them into one detection, causing temporary ID loss for one player.

**ID drift over time:** Over the 17-second clip, the same physical player could accumulate 2-4 different IDs due to repeated exits and re-entries at the frame boundary. This inflated the unique ID count from ~10 real players to 43 assigned IDs.

**Label readability:** At high player density (10-12 players), bounding box labels overlap significantly, reducing visual clarity in the annotated output.

---

## 6. Failure Cases Observed

- **Frame 375 and 393:** Player count dropped to 5, indicating some players moved outside the camera frame or were missed due to motion blur
- **High ID numbers (e.g., id:97, id:110):** These high IDs indicate players who re-entered the frame late in the video and received new track assignments
- **Partial detections at frame edges:** Players partially visible at the left and right edges of the frame were sometimes detected with lower confidence or missed entirely

---

## 7. Possible Improvements

**Short-term:**
- Use YOLOv8m or YOLOv8l for higher detection accuracy at the cost of speed
- Tune ByteTrack parameters (`track_high_thresh`, `track_buffer`) for the specific video characteristics
- Apply contrast enhancement or gamma correction as preprocessing for low-light video

**Long-term:**
- Integrate a ReID module (e.g., OSNet, BoT-SORT) to re-identify players who re-enter the frame using appearance features rather than position alone
- Fine-tune YOLOv8 on a sports-specific dataset such as SportsMOT or SoccerNet Tracking
- Implement team clustering using jersey color segmentation (HSV color histograms per bounding box)
- Add ball detection as a separate class for full game analytics

---

## 8. Bonus Features Implemented

- **Trajectory visualization:** Player movement paths overlaid on the court across all 424 frames, with unique colors per track ID
- **Player count over time chart:** Time-series visualization showing detection count per frame with average reference line (8.2 players/frame)

---

## 9. Conclusion

The pipeline successfully demonstrates real-world multi-object tracking on sports footage using modern, production-grade computer vision tools. YOLOv8 + ByteTrack provides a strong baseline for player detection and tracking with minimal configuration. The primary limitation — ID reassignment on re-entry — is a well-known open problem in multi-object tracking, addressable with ReID modules in future work.
