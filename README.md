# 💻🏋️ Real-Time AI Posture Correction for Bench Press Exercise Using MediaPipe and LSTM 🏋️💻
## 1️⃣ Abstract

This project develops a real-time system to monitor and correct Bench Press technique using Computer Vision. By leveraging **MediaPipe** for high-precision pose estimation and **LSTM (Long Short-Term Memory)** networks for temporal sequence analysis, the system classifies movement phases into four distinct categories: Correct Up/Down and Incorrect Up/Down. The goal is to provide immediate feedback to practitioners, helping to optimize muscle engagement and minimize the risk of injury during heavy lifting.

## 2️⃣ Project scope
- The study only evaluates exercise posture by calculating the degree of elbow extension relative to the body.
- Only applies to one person exercising within the camera's frame.
- The camera needs to be positioned parallel to the person exercising, so that their entire upper body and arms are clearly visible.
- Each lowering and raising motion needs to last approximately 1 second, corresponding to a camera resolution of 30 fps.
  <p align="center">
  <img width="299" height="538" alt="image" src="https://github.com/user-attachments/assets/b602fbed-7144-4bf7-81d3-a430b1fe4538" />
  <br>
  <em>Example of camera setup</em>
  </p>
## 3️⃣ State the problem
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
 
## 4️⃣ Method
### 4.1. MediaPipe Pose
  <p align="center">
  <img width="772" height="438" alt="MediaPipe" src="https://github.com/user-attachments/assets/4d2a0f83-326a-4ceb-919b-bae3cba424c4" />
  <br>
  </p>
    
- **Shoulder joint angle**: Let $A$ be the elbow, $B$ be the shoulder, and $C$ be the hip. The angle at the shoulder is determined using the vectors $\overrightarrow{BA}$ and $\overrightarrow{BC}$ according to the formula:

$$\cos(\theta) = \frac{\vec{BA} \cdot \vec{BC}}{|\vec{BA}| \times |\vec{BC}|}$$

- **Angular velocity**:

$$\theta = \theta_t - \theta_{t-1}$$

- **Feature vector**:

$$v = [Angle_{left}, Angle_{right}, \theta_{left}, \theta_{right}]$$

### 4.2. Normalization
- **Angle Normalization**:

$$\theta' = \frac{\theta - \theta_{min}}{\theta_{max} - \theta_{min}}$$

- **Velocity Normalization**:

$$Velocity' = \frac{Velocity}{180}$$

- **If a video segment exceeds 30 frames, the system uses `np.linspace` to select representative frames distributed evenly across the sequence.** `indices = np.linspace(0, len(temp_1_angles) - 1, window_size)`

### 4.3. LSTM Model
- **Input**: Feature sequence of shape $(Batch\_size, 30, 4)$.
- **LSTM Layer**: 1 layer with 32 hidden units.
- **Overfitting Mitigation**: A Dropout layer (rate: 0.2) is inserted to prevent the model from over-memorizing the training set.
- **Output**: The final layer classifies the input into 4 labels: Len_Dung(Up_Correct), Xuong_Dung(Down_Correct), Len_Sai(Up_Incorrect), Xuong_Sai(Down_Incorrect).
- **Softmax**: A Softmax function converts the final output into probabilities for each class.

### 4.4. Training process
The model is trained using the **Adam** optimizer with a learning rate of $0.0005$. The chosen loss function is **Sparse Categorical Crossentropy**, with **Accuracy** as the primary evaluation metric. To ensure stability and generalization, **K-Fold Cross Validation** is applied. In each fold:
- **Epochs**: 50
- **Batch size**: 16
- **Validation**: Data is drawn directly from the current fold.
The model with the highest validation accuracy across all folds is saved as the **Best Model**.
