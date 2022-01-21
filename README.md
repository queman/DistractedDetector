# DistractedDetector
Detection of distracted status (using Face Mesh)
# Problem
We proposed a method to detect the status of a participant in an online meeting/ class,... from real-time video (webcam), whether he is distracted or not?
# Method
1. [Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) (MediaPipe) is used to detect 468 3-D facial landmarks with fast and lightweight ultilization.
![facemesh_process drawio](https://user-images.githubusercontent.com/80699068/150458401-b817e7bc-8a12-4f89-b22c-d9582b642ebe.png)

2. Select the landmarks of interest for head, eyes, lip

3. Measure the deviation of facial attributes in each region compared with the ones of "attentive" status.
- Head's landmarks to detect head pose estimation (looking at another direction)
- Eyes' landmarks to detect aperture of eyes (sleepness, drowsiness), direction of eyes (looking to the left/right)
- Lip's landmarks to detect the distance from the upper lip to the lower lip (talking or yawning)
For this method, we have used the rule-based model according to Emotional AI rules of [Khan et al.,2020](http://dx.doi.org/10.5815/ijigsp.2020.02.03). It includes: 
- Behiavior rules: defining the range of deviation in which the behavior is determined as distracted or not, each of the rules is applied to one group of landmarks
- Threshold rules: defining the threshold of time for distracted behavior. If that behavior lasts longer than the threshold, then we finally conclude that he is distracted (there's no matter when we look somewhere awhile, isn't it?).

# Flow of logic
![Untitled Diagram_hot drawio](https://user-images.githubusercontent.com/80699068/150455792-94febdcf-bdad-4b53-8b84-594f304c07f2.png)


# Results
- In case of one participant whose face is in center region of camera with normal not too bad lighting condition, the Face Mesh + Rules method perform well in most situations (also capable when the face is slightly occluded or crossing the image boundary) for realtime-video.
- The speed of detection is about 20-30 FPS on my Intel(R) Core(TM) i7-3740QM CPU

# Evaluation
- Input : A  video from webcam of a participant in a conference
- Method: 
  - Manually annotate "Distracted"(1) or  "No distracted" (0) or "No face detected"(2) for 1 frame after every 20 frames (~1 frame/s). For simplicity, I ignore the threshold rules while evaluating. So each frame is labeled as "
  - Compare the annotation with the results from FaceMesh + Emotional AI rules' distraction detector
  - Accuracy = Total number of frames were correctly predicted by model  /   Total number of frames 
  - The wrong detection is considered to be the misclassification between "1" and "0" statuses or when more than half of the face is shown but get "2" label.
 - Here is my evaluation on some of the meeting recordings (all with pixel sizes 352 x 288) from [AMI Meeting Corpus](https://groups.inf.ed.ac.uk/ami/corpus/overview.shtml):

|                    | ES2002a.Closeup1.avi | ES2016a.Closeup3.avi | EN2003a.Closeup2.avi | EN2003a.Closeup1.avi | IB4003.Closeup1.avi | ES2016a.Closeup1.avi |                            |
|--------------------|----------------------|----------------------|----------------------|----------------------|---------------------|----------------------|----------------------------|
|  length            | 20m30s               | 23m4s                | 37m19s               | 37m17s               | 33m38s              | 23m3s                |                            |
| # frames           | 1537                 | 1730                 | 2799                 | 2796                 | 2523                | 1729                 |                            |
| # wrong detections | 79                   | 458                  | 573                  | 84                   | 162                 | 175                  |                            |
| Accuracy           | 0.9486               | 0.7352               | 0.7953               | 0.9700               | 0.9358              | 0.8988               | Average accuracy = 0.8806  |
   
   The accuracies exhibit at different levels. Most of the wrong cases are due to:
   - Wrong detections of eye direction. The BINARY_THRESHOLD algorithm used here are sensitive to pixel values. Therefore, our model can't recognize when one's eyes looking at left/right in bad light condition or in low quality videos; when eyes are small,... 
![frame548](https://user-images.githubusercontent.com/80699068/150468654-1daa1830-9504-4918-bdfd-f92f0a8eeb00.jpg)

![frame545](https://user-images.githubusercontent.com/80699068/150468680-cb204f27-f0f3-465d-bf3e-f0663ca3688b.jpg)


   - When the participant 
   - 
