# 💻 Real-Time AI Posture Correction for Bench Press Exercise Using MediaPipe and LSTM 💻
## 1️⃣ Project scope
- The study only evaluates exercise posture by calculating the degree of elbow extension relative to the body.
- Only applies to one person exercising within the camera's frame.
- The camera needs to be positioned parallel to the person exercising, so that their entire upper body and arms are clearly visible.
- Each lowering and raising motion needs to last approximately 1 second, corresponding to a camera resolution of 30 fps.
  <p align="center">
  <img width="299" height="538" alt="image" src="https://github.com/user-attachments/assets/b602fbed-7144-4bf7-81d3-a430b1fe4538" />
  <br>
  <em>Example of camera setup</em>
  </p>
## 2️⃣ State the problem
### 2.1. Input Data

The system's input is a set of training samples $(X, \hat{y})$ extracted from self-recorded videos. Each sample represents a **1-second video segment** corresponding to a single phase of the movement (e.g., the eccentric or concentric phase). The data is represented as a time-series sequence $X$, consisting of $n$ consecutive feature frames:

$$X = (f_1, f_2, f_3, ..., f_n)$$

- **Where:**
    * $n = 30$: sequence length, corresponding to **30 frames per second (fps)**, covering a full 1-second duration of the exercise phase.
    * $f_t$: feature vector at time $t$ ($1 \le t \le n$), containing the pose landmarks of the practitioner extracted at each frame.
- **Label ($\hat{y}$):** Each sequence $X$ is assigned a target label $\hat{y}$ that classifies the specific state of the action (e.g., "Down/Descend" or "Up/Ascend") as predefined in the dataset.

### 2.2. Output Data

The system's output is the predicted label for the movement's state at the current time step:

$$\hat{y} \in \lbrace L_1, L_2, L_3, L_4 \rbrace$$

* **With the corresponding values:**
    * **$L_1$ (Correct_Up):** The upward phase (concentric) performed with correct form.
    * **$L_2$ (Correct_Down):** The downward phase (eccentric) performed with correct form.
    * **$L_3$ (Incorrect_Up):** The upward phase performed with technical errors.
    * **$L_4$ (Incorrect_Down):** The downward phase performed with technical errors.
 
## 3️⃣ Method

