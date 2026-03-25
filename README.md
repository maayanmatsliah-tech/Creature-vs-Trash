# Trash vs. Marine Life Classifier

A deep learning image classifier that distinguishes between ocean trash/debris and marine life, built to support automated ocean pollution detection systems.

🔗 **Live Demo:** *Coming soon*

---

## Motivation

Marine debris is one of the most pressing environmental challenges facing ocean ecosystems today. Manual identification and removal of underwater trash is time-consuming and expensive. This project explores whether a convolutional neural network can reliably distinguish between ocean trash and marine life — a foundational step toward building automated debris detection and collection systems.

---

## Dataset

The model was trained on two combined datasets:

### Sea Animals
- **Source:** [Sea Animals Dataset](https://www.kaggle.com) (Kaggle)
- **Size:** ~19,000 images
- **Classes:** Various marine life (fish, jellyfish, coral, etc.)

### Ocean Trash / Marine Debris
- **Source 1:** Original project dataset
- **Source 2:** [UW Garbage Debris Dataset](https://www.kaggle.com/datasets/siddharth2305ego/underwater-garbagedebris) (Kaggle) — 5,130 underwater debris images

### Final Split
| Split | Sea Animals | Trash | Total |
|-------|-------------|-------|-------|
| Train | 15,214 | 5,575 | 20,789 |
| Val   | 3,804  | 1,394 | 5,198  |
| **Total** | **19,018** | **6,969** | **25,987** |

> **Note on class imbalance:** The dataset has an approximate 3:1 ratio of sea animals to trash. This was a deliberate improvement from the original 10:1 ratio, achieved by adding the UW Garbage Debris dataset. Further balancing is noted as a future improvement.

---

## Model

### Architecture
- **Base model:** ResNet-18 (pretrained on ImageNet)
- **Technique:** Transfer learning — all layers fine-tuned, final fully connected layer replaced with a 2-class output layer
- **Framework:** PyTorch

### Training Configuration
| Parameter | Value |
|-----------|-------|
| Optimizer | SGD (momentum=0.9) |
| Learning rate | 0.001 |
| LR scheduler | StepLR (step=7, gamma=0.1) |
| Batch size | 4 |
| Epochs | 10 |
| Hardware | NVIDIA T4 GPU (Google Colab) |

### Why Transfer Learning?
ResNet-18 was pretrained on ImageNet's 1.2 million images, giving it strong low-level feature representations (edges, textures, shapes). Fine-tuning this model on our domain-specific dataset allows us to leverage that prior knowledge while adapting to the specific visual characteristics of underwater imagery.

---

## Results

> ⚠️ *Results below are from the initial training run (before dataset balancing). Updated metrics will be added after retraining.*

### Classification Report
| Class | Precision | Recall | F1-Score | Support |
|-------|-----------|--------|----------|---------|
| Sea Animals | 0.90 | 0.45 | 0.60 | 3,804 |
| Trash | 0.08 | 0.48 | 0.13 | 368 |
| **Accuracy** | | | **0.46** | **4,172** |

### Confusion Matrix
*[Insert confusion matrix image here]*

---

## Error Analysis

The initial model showed signs of significant class bias due to the 10:1 imbalance in the training data:

- **High trash recall (0.48) but very low precision (0.08):** The model was triggering "trash" predictions too broadly, misclassifying many sea animals as trash.
- **Low sea animal recall (0.45):** Despite having far more sea animal training examples, the model still struggled — suggesting it defaulted to predicting trash in uncertain cases.
- **Overall accuracy of 46%** is close to random chance, indicating the model had not learned meaningful discriminating features and was likely exploiting class frequency patterns rather than visual content.

### Root Cause
The primary issue was **severe class imbalance** — approximately 10 sea animal images for every 1 trash image. The model learned that predicting "sea animal" was almost always correct, leading to poor generalization on the minority class.

### Fix Applied
Added ~5,130 underwater debris images from the UW Garbage Debris dataset, reducing the imbalance from 10:1 to approximately 3:1. The model was then retrained from scratch.

---

## How to Run

### Requirements
```
torch
torchvision
numpy
matplotlib
seaborn
scikit-learn
```

### Training
1. Clone this repository
2. Organize your dataset in the following structure:
```
datasets/
├── train/
│   ├── sea_animals/
│   └── trash/
└── val/
    ├── sea_animals/
    └── trash/
```
3. Open `ocean_life_vs_trash.ipynb` in Google Colab
4. Mount your Google Drive and update `data_dir` to your dataset path
5. Run all cells in order

### Inference
```python
import torch
from torchvision import models, transforms
from PIL import Image

# Load model
model = models.resnet18()
model.fc = torch.nn.Linear(model.fc.in_features, 2)
model.load_state_dict(torch.load('model.pt'))
model.eval()

# Preprocess image
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])
])

img = Image.open('your_image.jpg')
img_tensor = transform(img).unsqueeze(0)

# Predict
with torch.no_grad():
    output = model(img_tensor)
    _, pred = torch.max(output, 1)
    classes = ['sea_animals', 'trash']
    print(f"Prediction: {classes[pred.item()]}")
```

---

## Future Improvements

- **Further dataset balancing:** Increase trash images to match sea animal count (~19,000) for a 1:1 ratio
- **Data augmentation:** Apply stronger augmentation on the minority class (rotation, color jitter, flipping) to synthetically balance the dataset
- **Weighted loss function:** Use `CrossEntropyLoss` with class weights to penalize misclassification of the minority class more heavily
- **Larger architecture:** Experiment with ResNet-50 or EfficientNet for improved feature extraction
- **Deploy live demo:** Build and deploy a Hugging Face Spaces demo for real-time image classification

---

## Project Structure
```
├── ocean_life_vs_trash.ipynb   # Main training notebook
├── model.pt                    # Trained model weights
└── README.md                   # This file
```

---

## Author
Maayan Matsliah — Computer Science @ Northeastern University, AI Concentration