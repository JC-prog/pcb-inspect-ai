The project results demonstrate that the proposed automated framework successfully identifies detailed PCB structural features, with the **fine-tuned YOLOv12 model emerging as the optimal solution** for deployment in an Automated Optical Inspection (AOI) pipeline.

### **1. Semantic Segmentation Results**
The DeepLabV3_TransDecoder model achieved high precision in isolating PCB components:
*   **Accuracy:** Training achieved over **97.5% pixel accuracy** with stable convergence over 500 epochs.
*   **IoU Performance:** The model showed robust generalization on unseen data, achieving near-perfect Intersection-over-Union (IoU) scores (≈0.9–1.0) for component body regions.
*   **Observations:** Lead and solder regions showed moderate variability due to their higher structural complexity. Failure cases were primarily linked to **dense reflective leads** causing glare and **low-contrast leads** being underrepresented in the dataset.

### **2. Feature Detection Model Performance**
The study compared three architectures, focusing on Mean Average Precision (mAP) and recall:

| Metric | YOLOv12 (Fine-tuned) | RT-DETR | RF-DETR | Hybrid ConvXGB |
| :--- | :--- | :--- | :--- | :--- |
| **mAP@0.5** | **0.839** | 0.825 | 0.773 | 0.718 |
| **mAP@0.5:0.95** | 0.741 | 0.714 | 0.655 | - |
| **Precision** | 0.974 | 0.958 | **0.991** | N/A |
| **Recall** | **0.779** | 0.772 | 0.700 | N/A |
| **Epochs** | 160 (100 + 60) | 100 | 100 | N/A |
| **Training Speed** | **10-15 sec/epoch** | - | 3-10 min/epoch | N/A |

*   **YOLOv12 Observations:** Fine-tuning via a two-stage regime (heavy then light augmentation) significantly improved mAP from 0.68 to 0.72. It is the recommended model because its **higher recall score** minimizes missed defects, which is critical for AOI.
*   **RT-DETR Observations:** Evaluated as an intermediate candidate during model selection. The team trialled RT-DETR but ultimately narrowed down to YOLOv12 due to its superior recall and significantly faster training speed for this task.
*   **RF-DETR Observations:** Trained to 100 epochs, achieving mAP@0.5 of 0.773 and the highest precision (0.991) across all models. However, its recall of 0.700 indicates more conservative predictions, leading to a higher number of false negatives compared to YOLOv12 - a critical drawback for AOI applications.
*   **Hybrid ConvXGB Observations:** The inclusion of XGBoost for classification did not improve overall mAP but did increase prediction confidence for majority classes. It struggled with the **"Lead" class** due to data imbalance and its visual similarity to solder paste.

### **3. Key Observations and Error Analysis**
*   **Annotation Efficiency:** Training with **bounding boxes** led to higher mAP (0.68 vs. 0.65) and faster convergence compared to polygon annotations, as boxes align more directly with the model's regression objective.
*   **Common Failure Modes:** Both YOLO and RF-DETR frequently misclassified **sticker labels** as component bodies or solder paste. This occurred because stickers were not part of the initial training classes and share similar rectangular geometries with components.
*   **Full-Board Inference:** Standard resizing of high-resolution boards caused significant information loss, making small components undetectable. The team utilized **Slicing Aided Hyper Inference (SAHI)** to maintain resolution, though they noted it is not yet suitable for real-time production due to increased processing time.