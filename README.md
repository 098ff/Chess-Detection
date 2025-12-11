# ♟️ Automated Chess PGN Generator from Video Stream

## 1\. Overview

This project develops a complete computer vision pipeline to automatically detect chess pieces, identify their positions, and convert the entire game's sequence of moves into a standardized **Portable Game Notation (PGN)** file from a given video input.

## 2\. Problem Statement

To convert complex dynamic image data (chess videos with varying angles, changing lighting, and hand obscuration) into structurally accurate, rule-compliant PGN data that adheres to strict recording formats.

## 3\. Methodology & Pipeline

The system is divided into 4 core sequential steps:

### A. Board Detection & Perspective Correction (Classical Vision)

  * **Objective:** To accurately locate the chessboard boundaries and transform the image into a perfect 2D top-down view.
  * **Technique:** **Hough Transform** for line detection, followed by **Clustering** to identify the 4 corner points, which are then used to calculate the **Perspective Transform Matrix**.
  * **Pros & Cons:**
      * **Pro:** High processing speed on CPU, making it suitable for real-time operation.
      * **Con:** Highly sensitive to shadows and glare, which can introduce "false edges" and requires precise parameter tuning.

### B. Piece Detection & Classification (Deep Learning)

  * **Objective:** To localize and classify every piece on the corrected 2D board.
  * **Technique:** **YOLOv8n** model, Fine-tuned to classify 12 piece types.
  * **Training Strategy:** The model was trained in two stages. The second training round included **Negative Samples (empty board backgrounds)** to boost the model's robustness and significantly increase the **Recall** metric by improving background-object separation.

### C. State Change Analysis (Game Logic)

  * **Objective:** Detect the change in the board state between video frames to deduce the actual chess move.
  * **Technique:** Utilizes the **Python-Chess** library to manage and compare the **Board State** (represented by FEN strings) before and after a move, allowing the system to validate complex moves like Castling, En Passant, and Pawn Promotion.

### D. PGN Generation & Formatting (Logic Fix)

  * **Objective:** Convert the validated move into the final, correctly formatted PGN Output.
  * **Challenge Solved:** Resolved issues related to **Logic Error/Format Mismatch** (e.g., misformatting move numbering or checks), which were critical for achieving a high **Edit Distance** score, by implementing a dedicated **PGN Formatting Logic Fix**.

## 4\. Evaluation & Results

Evaluation covers both quantitative metrics for piece detection and the final PGN output.

### A. Quantitative Evaluation (PGN and Aggregate Model Metrics)

| Metric | Description | Result (Best Model) |
| :--- | :--- | :--- |
| **Edit Distance** | Primary metric measuring the structural difference between predicted PGN and solution (Lower is better). | **0.48** |
| **mAP50 (Aggregate)** | Mean Average Precision at IoU 50% (Overall piece detection accuracy). | **0.9658** |
| **Recall (R)** | Completeness of piece detection (Model's ability to find all existing pieces). | **0.9406** |
| **Precision (P)** | Overall object detection Precision. | **0.9366** |

### B. Per-Class Metrics (Piece-specific Performance)

| Class | Precision (P) | Recall (R) | mAP50 (AP) | mAP50-95 (AP) | Conclusion |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Overall** | **0.937** | **0.941** | **0.966** | **0.842** | **Strong overall performance.** |
| white-rook | 0.944 | **0.998** | **0.992** | 0.880 | **Highest Recall.** Excellent detection completeness. |
| white-pawn | 0.950 | 0.993 | 0.992 | 0.851 | Very high Recall, robust detection. |
| white-queen | **0.979** | 0.929 | **0.990** | **0.878** | **Highest mAP50.** Near-perfect accuracy. |
| white-knight | 0.978 | 0.900 | 0.986 | 0.863 | Excellent accuracy and precision. |
| white-king | 0.900 | 1.000 | 0.987 | 0.857 | **Perfect Recall (1.000).** High accuracy. |
| black-bishop | 0.957 | 0.927 | 0.976 | 0.860 | High accuracy and balance. |
| black-pawn | 0.994 | 0.960 | 0.983 | 0.851 | Extremely high Precision, strong performance. |
| black-knight | 0.906 | **0.826** | **0.919** | **0.779** | **Most Challenging.** Lowest Recall and mAP50-95. |
| black-king | **0.760** | 1.000 | 0.866 | 0.751 | Low Precision, but detected every instance (Recall 1.000). |
| black-queen | 0.917 | 0.920 | 0.932 | 0.831 | Solid balance. |
| black-rook | 0.993 | 0.905 | 0.983 | 0.853 | High Precision. |

### C. Qualitative Evaluation

  * **Confidence Score:** The model consistently achieved a high detection confidence score up to **0.94** for individual pieces.
  * **Confusion Matrix:** Training strategy successfully boosted overall **Recall (R)** from **0.54** (First Model) to **0.94** (Second Model), validating the effectiveness of Negative Sample augmentation.

## 5\. Key Technologies

| Category | Tool / Library | Role |
| :--- | :--- | :--- |
| **Object Detection** | **YOLOv8** (Ultralytics) | Piece Classification & Localization |
| **Classical Vision** | OpenCV (Hough Transform, Clustering) | Board Detection, Perspective Transformation |
| **Game Logic** | **Python-Chess** | FEN/PGN Management, Move Validation |
| **Data Analysis** | Pandas, Editdistance | Evaluation and Score Calculation |
