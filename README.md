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

The final model was trained for 10 epochs on 16,685 training images and 4,172 validation images, after applying the class-imbalance fix described above.

### Training Progress
| Epoch | Train Loss | Train Acc | Val Loss | Val Acc |
|-------|-----------|-----------|----------|---------|
| 0 | 0.0722 | 0.9760 | 0.0231 | 0.9919 |
| 1 | 0.0477 | 0.9850 | 0.0186 | 0.9930 |
| 2 | 0.0293 | 0.9896 | 0.0172 | 0.9954 |
| 3 | 0.0266 | 0.9909 | 0.0188 | 0.9942 |
| 4 | 0.0176 | 0.9938 | 0.0099 | 0.9962 |
| 5 | 0.0197 | 0.9931 | 0.0070 | 0.9969 |
| 6 | 0.0199 | 0.9942 | 0.0065 | 0.9978 |
| 7 | 0.0077 | 0.9977 | 0.0064 | 0.9976 |
| 8 | 0.0068 | 0.9980 | 0.0058 | 0.9978 |
| 9 | 0.0087 | 0.9975 | 0.0059 | 0.9981 |

**Best validation accuracy: 99.81%** (training time: 74m 22s on an NVIDIA T4 GPU)

> An earlier run on the original, more imbalanced dataset (~10:1) performed close to chance (~46% accuracy), which is what motivated the dataset rebalancing described above.

### Classification Report & Confusion Matrix
The notebook includes an evaluation cell (run after training) that computes a per-class classification report and confusion matrix on the validation set using `scikit-learn` and `seaborn`.

---

## How to Run

### Requirements
```
pip install -r requirements.txt
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
5. Run all cells in order — the trained weights are saved to `model.pt` in your dataset directory, and a classification report/confusion matrix are printed at the end

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
├── requirements.txt            # Python dependencies
├── LICENSE                     # MIT license
├── .gitignore                  # Excludes datasets/, model weights, caches
└── README.md                   # This file
```
`model.pt` (trained model weights) is generated by the notebook and is not tracked in this repository — see [How to Run](#how-to-run).

---

## Author
Maayan Matsliah — Computer Science @ Northeastern University, AI Concentration