# DistractedDetector
Detection of distracted status (using Face Mesh).

# Problem
We proposed a method to detect the status of a participant in an online meeting/ class,... from real-time video (webcam), whether he is distracted or not?

# Method

1. Use [Face Mesh](https://google.github.io/mediapipe/solutions/face_mesh.html) (MediaPipe) to detect 468 3-D facial landmarks. Face Mesh is fast and lightweight 
for real-time application.
  ![facemesh_process drawio](https://user-images.githubusercontent.com/80699068/150458401-b817e7bc-8a12-4f89-b22c-d9582b642ebe.png)

2. Select the landmarks of interest for head, eyes, lip: only 18 landmarks is chosen.

3. Measure the deviation of facial attributes in each region compared with the ones of "attentive" status.
- Head's landmarks to detect head pose estimation (looking at another direction).
- Eyes' landmarks to detect aperture of eyes (sleepness, drowsiness), direction of eyes (looking elsewhere).
- Lip's landmarks to detect the distance from the upper lip to the lower lip (talking or yawning).

  For this method, we have used the rule-based model according to Emotional AI rules of [Khan et al.,2020](http://dx.doi.org/10.5815/ijigsp.2020.02.03). It includes: 
- Behiavior rules: defining the range of deviation in which the behavior is determined as distracted or not, each of the rules is applied to one group of landmarks
- Threshold rules: defining the threshold of time for distracted behavior. If that behavior lasts longer than the threshold, then we finally conclude that he is distracted (there's no matter when we look somewhere awhile, isn't it?).

  Note: We can modify the rules so that it suits in different situations (for example, in our demo, we treat the "Looking Down" detection more specially by setting its threshold time longer than other directions (Right, Left, Up) in case the participant is taking note or reading book,...).

# Flow of logic
![Untitled Diagram_hot drawio](https://user-images.githubusercontent.com/80699068/150455792-94febdcf-bdad-4b53-8b84-594f304c07f2.png)


# Results
- In case of one participant whose face is in center region of camera with normal not too bad lighting condition, the Face Mesh + Rules method performs well in most situations (also capable when the face is slightly occluded or crossing the image boundary) for real-time video.
- The speed of detection is about 20-30 FPS on our Intel(R) Core(TM) i7-3740QM CPU for input from webcam.

# Evaluation
- Input : A  video of a participant in a conference
- Method: 
  - Manually annotate "Distracted"(1) or  "No distracted" (0) or "No face detected"(2) for 1 frame after every 20 frames (~1 frame/s). For simplicity, we ignore the threshold rules while evaluating. Then each frame is labeled as "1" immediately when there is a distracted behevior detected (constrainted time = 0 for all attributes).
  - Compare the annotation with the results from FaceMesh + Emotional AI rules' distraction detector
  - Accuracy = Total number of frames were correctly predicted by model  /   Total number of frames 
  - The wrong detection is considered to be the misclassification between "1" and "0" statuses or when more than half of the face is shown but get "2" label.
 - Here is our evaluation on some of the meeting recordings (all with pixel sizes of 352 x 288) from [AMI Meeting Corpus](https://groups.inf.ed.ac.uk/ami/corpus/overview.shtml):

|                    | ES2002a.Closeup1.avi | ES2016a.Closeup3.avi | EN2003a.Closeup2.avi | EN2003a.Closeup1.avi | IB4003.Closeup1.avi | ES2016a.Closeup1.avi |                            |
|--------------------|----------------------|----------------------|----------------------|----------------------|---------------------|----------------------|----------------------------|
|  length            | 20m30s               | 23m4s                | 37m19s               | 37m17s               | 33m38s              | 23m3s                |                            |
| # frames           | 1537                 | 1730                 | 2799                 | 2796                 | 2523                | 1729                 |                            |
| # wrong detections | 79                   | 458                  | 573                  | 84                   | 162                 | 175                  |                            |
| Accuracy           | 0.9486               | 0.7352               | 0.7953               | 0.9700               | 0.9358              | 0.8988               | Average accuracy = 0.8806  |
   
   The accuracies exhibit at different levels. It can mostly detect the distracted/ no distracted status correcly when the face actually does so. The wrong cases come from the misclassification, i.e. detect "1" while "0" is true and vice versa.
   
   - Wrong detections of eye direction:
  
     + The BINARY_THRESHOLD algorithm used here is sensitive to the pixel values. Therefore, our method can't recognize when one's eyes looking at left/right in bad light condition or in low quality videos; when eyes are small,... 
     
![frame548](https://user-images.githubusercontent.com/80699068/150468654-1daa1830-9504-4918-bdfd-f92f0a8eeb00.jpg)
![frame545](https://user-images.githubusercontent.com/80699068/150468680-cb204f27-f0f3-465d-bf3e-f0663ca3688b.jpg)
![frame1402](https://user-images.githubusercontent.com/80699068/150479926-b6e82ec0-cc62-462b-a94b-4ff36e5c0008.jpg)
![frame229](https://user-images.githubusercontent.com/80699068/150480264-fed75e0f-cc0c-4893-b057-36ec698ca4e4.jpg)

     + When the participant's pose is towards another direction (from the center) but his eyes are shown to concentrate to the center indeed. Our method sees it as a "looking elsewhere" case.
     
![frame923](https://user-images.githubusercontent.com/80699068/150478029-b0c67e6a-736f-4529-ac38-9a24b7cbeaaf.jpg)
![frame955](https://user-images.githubusercontent.com/80699068/150478051-652362c3-afec-483f-8fa5-9e58d57e933d.jpg)
![frame1989](https://user-images.githubusercontent.com/80699068/150478255-70e2ef3b-56d7-461f-b92a-18fb6f26952c.jpg)
![frame96](https://user-images.githubusercontent.com/80699068/150479700-2d41d78a-f5d5-4128-b22f-2be4e4688de9.jpg)
![frame176](https://user-images.githubusercontent.com/80699068/150479834-510e5150-ec5a-430d-bff7-a371626a260a.jpg)
![frame1551](https://user-images.githubusercontent.com/80699068/150480113-16c189e0-1bdd-4cf5-b2e4-38dc14df0162.jpg)


   - Wrong "No face detected": Our model can not detect the face when it is obscured for the most part. We consider the detection is wrong when it can't detect an image with more than half of the face is shown.


![frame1251](https://user-images.githubusercontent.com/80699068/150479378-fc1dc539-f436-47e4-812d-7241d7b6c772.jpg)
![frame2345](https://user-images.githubusercontent.com/80699068/150479418-11c516e7-0449-424c-bddc-02dd93e32144.jpg)
![frame53](https://user-images.githubusercontent.com/80699068/150479547-027cfe79-ff16-426b-ac71-14b793654d7c.jpg)
![frame311](https://user-images.githubusercontent.com/80699068/150480649-fff09ebc-dfc8-4549-ab42-470d9fcad2db.jpg)

Though the above accuracies are not consistent for various cases. It is somehow because of our way in evaluation (the time dimension is ignored). After all, its ability in real-time application is good (see a demo in example)

# Future plans:
- We'd like to handle the "eye direction" error better by more accurate method of eyes detection (Iris solution of MediaPipe may help)
- Experiment more by apply the model-based method (with machine learning/ deep learning models) instead of the rule-based one
- Widen the dataset for evaluation
- Deploy the detector as a web API for real-time input which can also send backs the statiscal results of the participant's attention level during the meeting/ lecture. 
