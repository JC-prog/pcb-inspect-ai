The project’s methodology focuses on creating a fully automated PCB inspection pipeline that moves from raw image acquisition to pixel-level feature extraction and object localization. The core framework is divided into data preparation, semantic segmentation, and advanced object detection.

### **1. Data Preparation and Preprocessing**
*   **Collection & Tiling:** High-resolution RGB images are acquired using an MRS sensor integrated into an AOI system. To preserve fine-grained detail without overwhelming computational resources, full PCB panels are tiled into **$800 \times 800$ pixel patches**.
*   **Annotation Pipeline:** The project utilizes manually labeled, color-coded segmentation masks. An automated Python pipeline transforms these dense masks into **COCO-compatible bounding-box labels** for object detection, ensuring consistency and eliminating manual box drawing.
*   **Augmentation:** To improve generalization, the dataset undergoes transformations including random horizontal/vertical flips, rotations, scaling, and HSV brightness adjustments.

### **2. Semantic Segmentation (DeepLabV3_TransDecoder)**
The segmentation stage aims to produce high-resolution pixel predictions for three classes: Component Body, Lead, and Solder Paste.
*   **Architecture:** It combines a **DeepLabV3 encoder** (using a ResNet-50 backbone) with a **Transformer-inspired cross-attention decoder**.
*   **Feature Fusion:** The decoder uses **Cross-Attention Sampling** to allow high-resolution spatial tokens to attend to deep semantic context, improving edge accuracy.
*   **Multi-Scale Context:** An **Atrous Spatial Pyramid Pooling (ASPP)** module captures features at various scales using dilated convolutions with multiple atrous rates.

### **3. Feature Detection Models**
The project evaluates and iterates on three distinct detection architectures:
*   **YOLOv12 (One-Stage Detector):** Incorporates a **FlashAttention Area Attention (A2) module**, which splits feature maps into quadrants to reduce computational complexity and improve accuracy at high resolutions.
*   **RT-DETR (Real-Time Detection Transformer):** Baidu's end-to-end transformer detector evaluated as a candidate architecture. Uses a hybrid encoder combining intra-scale feature interaction and cross-scale feature fusion. Evaluated during the model selection phase before the team narrowed down to YOLOv12 as the primary detector.
*   **RF-DETR (Transformer-based):** Utilizes a **DINOv2 backbone** (a self-supervised pretrained transformer) to achieve faster convergence and high precision. It employs multi-scale deformable attention in the encoder to extract features near specific reference points.
*   **Hybrid ConvXGB:** A sequential pipeline where a YOLOv12 model extracts deep feature embeddings (truncated before the final detection head) and passes them to an **XGBoost classifier**. This ensemble of decision trees is used to refine final defect labels.

### **4. Training and Fine-Tuning Strategy**
For optimal performance, the recommended **YOLOv12** model follows a two-stage training regime:
*   **Stage 1 (Fast Pretraining, 100 epochs):** Focuses on rapid convergence on foundational PCB patterns using $640 \times 640$ resolution images with heavy augmentation.
*   **Stage 2 (Fine-Tuning Polish, 60 epochs):** Refines accuracy using higher resolution ($896 \times 896$) and lighter data augmentation to stabilize the model for final deployment. Total: 160 epochs.

### **5. Evaluation and Inference**
*   **Metrics:** Models are evaluated based on **Mean Average Precision (mAP@0.5 and mAP@0.5:0.95)**, recall, and inference speed (FPS).
*   **Full-Board Inference:** For inspection of full-sized boards, the project implements **Slicing Aided Hyper Inference (SAHI)**, which performs inference on overlapping tiles and stitches the results back together to prevent information loss from resizing.
*   **Application:** A web-based Gradio interface was developed to allow users to upload images, select between architectures, and export prediction data in JSON format.