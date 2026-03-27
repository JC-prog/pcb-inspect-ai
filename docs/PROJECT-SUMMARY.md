This project, titled **Automated Generation of PCB Inspection Recipes Using Deep Learning-Based Feature Detection**, focuses on automating the quality control process in semiconductor manufacturing. The primary goal is to replace the manual, labor-intensive process of teaching Automated Optical Inspection (AOI) systems with an automated framework that leverages deep learning to identify detailed structural PCB features.

### **Project Overview**
The project addresses limitations in current AOI recipe generation, which relies on CAD and Bill of Materials (BoM) data but lacks the fine-grained structural information required for precise inspection. The proposed solution is an end-to-end pipeline that performs semantic segmentation followed by object detection to extract critical parameters like pad size, lead width, and solder mask offset.

### **Key Components & Methodology**
The project architecture is divided into two primary deep learning stages:

1.  **Semantic Segmentation (DeepLabV3_TransDecoder):**
    *   **Architecture:** Combines a DeepLabV3 encoder (ResNet-50 backbone) with a Transformer-inspired cross-attention decoder.
    *   **Function:** Processes high-resolution RGB images to produce pixel-level maps for three classes: **Component Body, Lead, and Solder Paste**.
    *   **Performance:** Achieved over **97.5% pixel accuracy** and near-perfect IoU (0.9–1.0) for component body regions.

2.  **Feature Detection (YOLOv12, RT-DETR, RF-DETR):**
    *   **YOLOv12:** Incorporates advanced features like FlashAttention Area Attention (A2) for efficient parallel processing.
    *   **RT-DETR:** Baidu's Real-Time Detection Transformer evaluated as an intermediate candidate before narrowing down to YOLOv12.
    *   **RF-DETR:** A transformer-based detector utilizing a DINOv2 backbone for rapid convergence.
    *   **Hybrid ConvXGB:** A multi-stage model combining CNN feature extraction with an XGBoost classifier for refined defect labeling.

### **Results & Recommendations**
*   **Optimal Model:** The **fine-tuned YOLOv12** is the recommended model for deployment. It demonstrated a superior **recall score of 0.77**, which is critical in AOI for minimizing false negatives (missed defects).
*   **Efficiency:** YOLOv12 proved highly computationally efficient, completing training epochs in **10–15 seconds**, significantly faster than the 3–10 minutes required by RF-DETR.
*   **Generalization:** The model successfully localized and identified intricate components across unseen PCB datasets.

### **Application & Future Work**
*   **Web Application:** A functional comparison platform was developed using **Gradio**, allowing users to upload PCB images, run inference across different models, and export results (bounding box coordinates and confidence scores) in JSON format.
*   **Full-Board Inference:** For full-sized PCB inspection, the project utilizes **Slicing Aided Hyper Inference (SAHI)** to maintain resolution without information loss from resizing.
*   **Future Goals:** Improving robustness against reflective/specular surfaces and explicitly training the models to differentiate non-component "sticker labels" to further reduce false positives.

### **Tech Stack**
*   **Frameworks:** PyTorch, Ultralytics (YOLO), Roboflow (RF-DETR).
*   **Interface:** Gradio, Tkinter.
*   **Processing:** OpenCV, NumPy, SAHI.
*   **Deployment:** ONNX for model portability.