# Generative AI Cheatsheet

Your ultimate quick reference guide covering the full stack behind modern generative and machine learning systems — Python, math foundations, classical ML, deep learning, computer vision, NLP/transformers, training tricks, optimization, and production MLOps.

## 1️⃣ Python Essentials

List Comprehensions

```python
[x**2 for x in range(10)]
[x for x in range(20) if x % 2 == 0]
[[i*j for j in range(3)] for i in range(3)]
{x: x**2 for x in range(5)}  # dict comprehension
```

Lambda & Map/Filter

```python
add = lambda x, y: x + y
list(map(lambda x: x**2, [1, 2, 3]))
list(filter(lambda x: x > 0, [-1, 2, -3, 4]))

from functools import reduce
reduce(lambda x, y: x + y, [1, 2, 3, 4])
```

Decorators

```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Time: {time.time()-start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

Context Managers

```python
with open('file.txt', 'r') as f:
    data = f.read()
```

Generators

```python
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b
```

## 2️⃣ NumPy & Pandas

NumPy Quick Reference

```python
import numpy as np

# Array Creation
np.array([1, 2, 3])                  # From list
np.zeros((3, 4))                     # 3x4 zeros
np.ones((2, 3))                      # 2x3 ones
np.eye(4)                            # 4x4 identity
np.arange(0, 10, 2)                  # [0,2,4,6,8]
np.linspace(0, 1, 5)                 # 5 points between 0-1
np.random.randn(3, 4)                # Normal distribution
np.random.rand(3, 4)                 # Uniform [0,1)

# Array Operations
arr.shape                            # (3,4)
arr.reshape(4, 3)                    # Reshape
arr.T                                # Transpose
arr.flatten()                        # To 1D
np.concatenate([a, b], axis=0)       # Stack vertically
np.hstack([a, b])                    # Horizontal stack
np.vstack([a, b])                    # Vertical stack

# Indexing & Slicing
arr[0, 1]                            # Element at (0,1)
arr[:, 0]                            # First column
arr[arr > 5]                         # Boolean indexing
arr[[0, 2], [1, 3]]                  # Fancy indexing

# Math Operations
np.sum(arr, axis=0)                  # Sum columns
np.mean(arr, axis=1)                 # Mean rows
np.std(arr)                          # Standard deviation
np.dot(a, b)                         # Matrix multiplication
a @ b                                # Matrix multiplication (Python 3.5+)
np.linalg.inv(arr)                   # Matrix inverse
np.linalg.eig(arr)                   # Eigenvalues/vectors
```

Pandas Quick Reference

```python
import pandas as pd

# DataFrame Creation
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
df = pd.read_csv('file.csv')
df = pd.read_excel('file.xlsx')

# Inspection
df.head(10)                          # First 10 rows
df.tail(5)                           # Last 5 rows
df.info()                            # Column types, null counts
df.describe()                        # Statistics
df.shape                             # (rows, cols)
df.columns                           # Column names
df.dtypes                            # Data types

# Selection
df['col']                            # Single column (Series)
df[['col1', 'col2']]                 # Multiple columns
df.loc[0]                            # Row by label
df.iloc[0]                           # Row by position
df.loc[0:5, 'col']                   # Rows 0-5, specific column
df[df['col'] > 5]                    # Boolean filtering

# Operations
df.drop('col', axis=1)               # Drop column
df.drop([0, 1], axis=0)              # Drop rows
df.dropna()                          # Drop rows with NaN
df.fillna(0)                         # Fill NaN with 0
df.groupby('col').mean()             # Group by and aggregate
df.sort_values('col', ascending=False)
df.merge(df2, on='key')              # Join DataFrames
df.apply(lambda x: x**2)             # Apply function
df['new_col'] = df['A'] + df['B']    # Create column
```

## 3️⃣ Math Foundations

**Linear Algebra**

- Vector operations:
  - Dot product: `a·b = Σ(aᵢbᵢ)`
  - Norm: `‖a‖ = √(Σaᵢ²)`
  - Cosine similarity: `cos(θ) = (a·b)/(‖a‖‖b‖)`
- Matrix operations:
  - `(AB)ᵀ = BᵀAᵀ`
  - `(AB)⁻¹ = B⁻¹A⁻¹`
  - `det(AB) = det(A)·det(B)`
  - Eigenvalue equation: `Av = λv`

**Calculus**

- Derivatives:
  - `d/dx(xⁿ) = nxⁿ⁻¹`
  - `d/dx(eˣ) = eˣ`
  - `d/dx(ln x) = 1/x`
  - Chain rule: `(f∘g)' = f'(g)·g'`
- Gradient: `∇f = [∂f/∂x₁, ∂f/∂x₂, ..., ∂f/∂xₙ]`
- Hessian: `H = [∂²f/∂xᵢ∂xⱼ]`

**Probability**

- Basics:
  - `P(A∪B) = P(A) + P(B) - P(A∩B)`
  - `P(A|B) = P(A∩B) / P(B)`
  - Bayes' theorem: `P(A|B) = P(B|A)P(A)/P(B)`
- Distributions:
  - Normal: `N(μ, σ²)`
  - Bernoulli: `p(x=1) = p`
  - Binomial: `P(k) = C(n,k)·pᵏ·(1-p)ⁿ⁻ᵏ`

**Statistics**

- Mean: `μ = (1/n)Σxᵢ`
- Variance: `σ² = (1/n)Σ(xᵢ-μ)²`
- Standard deviation: `σ = √σ²`
- Covariance: `Cov(X,Y) = E[(X-μₓ)(Y-μᵧ)]`
- Correlation: `ρ = Cov(X,Y)/(σₓσᵧ)`

## 4️⃣ Machine Learning Algorithms

| Algorithm | Type | Formula / Key Concept | Use Case |
|---|---|---|---|
| Linear Regression | Supervised | `ŷ = wᵀx + b`, minimize MSE | Continuous prediction |
| Logistic Regression | Supervised | `σ(z) = 1/(1+e⁻ᶻ)`, binary classification | Binary classification |
| Decision Trees | Supervised | Split on Gini/Entropy, greedy | Interpretable, non-linear |
| Random Forest | Ensemble | Bagging of decision trees | High accuracy, robust |
| Gradient Boosting | Ensemble | Sequential tree training (e.g. XGBoost) | Competitions, tabular data |
| SVM | Supervised | Max margin, kernel trick | Classification, small data |
| K-Means | Unsupervised | Minimize within-cluster variance | Clustering |
| PCA | Unsupervised | Eigenvalue decomposition, reduce dims | Dimensionality reduction |
| K-NN | Supervised | Majority vote of k nearest neighbors | Simple baseline |

Scikit-learn Quick Template

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Scale features
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Predict
y_pred = model.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred))
```

## 5️⃣ Deep Learning

**Key Formulas**

- Forward pass: `z = Wx + b`, `a = σ(z)`
- Loss (cross-entropy): `L = -Σ(y·log(ŷ))`
- Backpropagation: `∂L/∂W = ∂L/∂a · ∂a/∂z · ∂z/∂W`
- SGD update: `W ← W - η∇L`
- Adam update: `m = β₁m + (1-β₁)g`, `v = β₂v + (1-β₂)g²`, `W ← W - η·m/√(v+ε)`

**Activation Functions**

- Sigmoid: `σ(x) = 1/(1+e⁻ˣ)`
- Tanh: `tanh(x) = (eˣ-e⁻ˣ)/(eˣ+e⁻ˣ)`
- ReLU: `f(x) = max(0, x)`
- Leaky ReLU: `f(x) = max(0.01x, x)`
- GELU: `f(x) = x·Φ(x)`
- Swish: `f(x) = x·σ(x)`

**Loss Functions**

- MSE: `(1/n)Σ(y-ŷ)²`
- MAE: `(1/n)Σ|y-ŷ|`
- BCE: `-[y·log(ŷ) + (1-y)·log(1-ŷ)]`
- CCE: `-Σy·log(ŷ)`
- Focal loss: `-(1-p)ᵞ·log(p)`
- Hinge: `max(0, 1-y·ŷ)`

PyTorch Neural Network Template

```python
import torch
import torch.nn as nn
import torch.optim as optim

class NeuralNet(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super(NeuralNet, self).__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.5)
        self.fc2 = nn.Linear(hidden_size, num_classes)

    def forward(self, x):
        out = self.fc1(x)
        out = self.relu(out)
        out = self.dropout(out)
        out = self.fc2(out)
        return out

# Training Loop
model = NeuralNet(784, 128, 10).to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

for epoch in range(num_epochs):
    for i, (images, labels) in enumerate(train_loader):
        images = images.reshape(-1, 784).to(device)
        labels = labels.to(device)

        # Forward pass
        outputs = model(images)
        loss = criterion(outputs, labels)

        # Backward and optimize
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {loss.item():.4f}')
```

## 6️⃣ Computer Vision

**CNN Layers**

- Conv2D output size: `⌊(W-K+2P)/S⌋+1` — `W`: input size, `K`: kernel size, `P`: padding, `S`: stride
- MaxPool2D: downsample by max, typically 2x2 with stride 2
- BatchNorm: `x̂ = (x-μ)/√(σ²+ε)`

**Architectures**

- LeNet: Conv → Pool → Conv → Pool → FC
- AlexNet: 5 conv layers, dropout, ReLU
- VGG: 3x3 conv stacks, very deep
- ResNet: Skip connections, 50-152 layers
- Inception: Multi-scale filters
- EfficientNet: Compound scaling
- ViT: Pure transformer for vision

**Tasks**

- Classification: image → label
- Detection: image → bounding boxes + labels
- Segmentation: image → pixel masks
- Instance segmentation: per-instance masks
- Pose estimation: keypoints
- Tracking: objects across frames
- GANs: image generation

PyTorch CNN

```python
class CNN(nn.Module):
    def __init__(self):
        super(CNN, self).__init__()
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, padding=1)
        self.bn1 = nn.BatchNorm2d(32)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.bn2 = nn.BatchNorm2d(64)
        self.pool = nn.MaxPool2d(2, 2)
        self.fc1 = nn.Linear(64 * 8 * 8, 128)
        self.fc2 = nn.Linear(128, 10)
        self.dropout = nn.Dropout(0.5)

    def forward(self, x):
        x = self.pool(F.relu(self.bn1(self.conv1(x))))
        x = self.pool(F.relu(self.bn2(self.conv2(x))))
        x = x.view(-1, 64 * 8 * 8)
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)
        return x
```

Transfer Learning

```python
from torchvision import models

model = models.resnet50(pretrained=True)
for param in model.parameters():
    param.requires_grad = False

model.fc = nn.Linear(2048, num_classes)  # Replace final layer
```

## 7️⃣ NLP & Transformers

**Transformer Components**

- Self-attention: `Attention(Q,K,V) = softmax(QKᵀ/√dₖ)V`
- Multi-head attention: `MultiHead = Concat(head₁,...,headₕ)W^O`
- Position encoding: `PE(pos,2i) = sin(pos/10000^(2i/d))`, `PE(pos,2i+1) = cos(pos/10000^(2i/d))`
- Layer norm: `LN(x) = γ(x-μ)/σ + β`

**Key Architectures**

- BERT: Bidirectional, MLM + NSP pretraining
- GPT: Autoregressive, causal masking
- T5: Text-to-text, unified framework
- RoBERTa: BERT with optimized pretraining
- ELECTRA: Replaced token detection
- DeBERTa: Disentangled attention

**Tasks**

- Classification: sentiment, intent
- NER: named entity recognition
- QA: question answering
- Summarization: extractive/abstractive
- Translation: sequence-to-sequence
- Generation: text completion

HuggingFace Transformers

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
from transformers import Trainer, TrainingArguments

# Load model and tokenizer
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased", num_labels=2
)

# Tokenize
inputs = tokenizer(
    texts, padding=True, truncation=True, max_length=512, return_tensors="pt"
)

# Fine-tuning
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    learning_rate=2e-5,
    logging_dir="./logs",
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
)
trainer.train()

# Inference
outputs = model(**inputs)
predictions = torch.argmax(outputs.logits, dim=-1)
```

## 8️⃣ Training Tricks

**Regularization**

- L1: `|W|`, encourages sparse weights
- L2: `‖W‖²`, weight decay
- Dropout: randomly drop neurons
- DropConnect: randomly drop connections
- Early stopping: stop when validation loss rises
- Data augmentation: synthetic training data
- Label smoothing: soft targets
- Mixup: linear interpolation of samples/labels

**Optimization**

- SGD: `W ← W - η∇L`
- Momentum: `v = βv + ∇L`
- RMSprop: adaptive learning rate
- Adam: momentum + RMSprop
- AdamW: Adam with decoupled weight decay
- LARS: large-batch training
- LAMB: LARS + Adam
- Lookahead: two-optimizer cycle

**Learning Rate Schedules**

- Constant: fixed LR
- Step decay: drop every N epochs
- Exponential decay: `lr = lr₀·e^(-kt)`
- Cosine annealing: smooth decay
- Warmup: linear increase, then decay
- OneCycle: peak in the middle of training
- ReduceLROnPlateau: drop when metric stalls
- Cyclic LR: oscillate between bounds

**💡 Best Practices**

- Start with Adam optimizer (`lr=1e-3` or `3e-4`)
- Use batch normalization or layer normalization
- Apply gradient clipping (`max_norm=1.0`) for RNNs/Transformers
- Use Xavier/He initialization for weights
- Monitor both training and validation loss
- Use mixed precision training (FP16) for speed

## 9️⃣ Model Optimization

| Technique | Speedup | Quality Loss | Method |
|---|---|---|---|
| Quantization | 2-4x | 0-2% | FP32 → INT8/FP16 |
| Pruning | 1.5-3x | 0-1% | Remove small weights |
| Knowledge Distillation | 2-10x | 1-3% | Teacher → student |
| TensorRT | 2-5x | 0% | NVIDIA optimization |
| ONNX Runtime | 1.5-3x | 0% | Cross-platform |
| Model Fusion | 1.2-1.5x | 0% | Merge layers |
| Flash Attention | 2-4x (memory) | 0% | Efficient attention |

Quantization (PyTorch)

```python
import torch.quantization as quant

model_fp32 = YourModel()
model_fp32.eval()

# Dynamic quantization (post-training)
model_int8 = quant.quantize_dynamic(
    model_fp32,
    {nn.Linear, nn.LSTM},
    dtype=torch.qint8
)

# Static quantization
model_fp32.qconfig = quant.get_default_qconfig('fbgemm')
model_prepared = quant.prepare(model_fp32)
model_prepared(calibration_data)  # Calibrate
model_int8 = quant.convert(model_prepared)
```

Mixed Precision Training

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()
for data, target in train_loader:
    optimizer.zero_grad()

    with autocast():
        output = model(data)
        loss = criterion(output, target)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

## 🔟 Production MLOps

Model Serving — FastAPI

```python
from fastapi import FastAPI
import torch

app = FastAPI()
model = torch.load('model.pt')

@app.post("/predict")
def predict(data: dict):
    inputs = preprocess(data)
    outputs = model(inputs)
    return {"prediction": outputs}
```

Model Serving — Docker

```dockerfile
FROM python:3.9
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["uvicorn", "main:app"]
```

**Monitoring**

- Data drift: input distribution shift
- Model drift: performance degradation over time
- Latency: P50, P95, P99
- Throughput: requests/sec
- Error rate: failed predictions
- Resource usage: CPU, GPU, memory

**MLOps Pipeline**

```text
Data → Feature Engineering → Train → Validate → Test → Deploy → Monitor → Retrain
```

## 1️⃣1️⃣ Model Evaluation

| Metric | Formula | Use Case |
|---|---|---|
| Accuracy | `(TP+TN)/(TP+TN+FP+FN)` | Balanced classes |
| Precision | `TP/(TP+FP)` | Minimize false positives |
| Recall | `TP/(TP+FN)` | Minimize false negatives |
| F1 Score | `2·(P·R)/(P+R)` | Balance precision and recall |
| AUC-ROC | Area under the ROC curve | Binary classification |
| RMSE | `√[(1/n)Σ(y-ŷ)²]` | Regression |
| MAE | `(1/n)Σ|y-ŷ|` | Regression (robust) |
| R² Score | `1 - SS_res/SS_tot` | Regression quality |

Evaluation Metrics

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
from sklearn.metrics import roc_auc_score, confusion_matrix, classification_report

# Classification
accuracy = accuracy_score(y_true, y_pred)
precision = precision_score(y_true, y_pred, average='macro')
recall = recall_score(y_true, y_pred, average='macro')
f1 = f1_score(y_true, y_pred, average='macro')
auc = roc_auc_score(y_true, y_pred_proba)

# Confusion Matrix
cm = confusion_matrix(y_true, y_pred)
print(classification_report(y_true, y_pred))

# Cross-validation
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"CV Accuracy: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

## 1️⃣2️⃣ Quick Commands

PyTorch

```bash
pip install torch torchvision
pip install transformers datasets
```

```python
# Check GPU
torch.cuda.is_available()
torch.cuda.device_count()

# Save/Load
torch.save(model.state_dict(), 'model.pt')
model.load_state_dict(torch.load('model.pt'))

# Move to GPU
model = model.to('cuda')
inputs = inputs.cuda()
```

TensorFlow

```bash
pip install tensorflow
```

```python
# Check GPU
tf.config.list_physical_devices('GPU')

# Save/Load
model.save('model.h5')
model = tf.keras.models.load_model('model.h5')

# Mixed precision
policy = tf.keras.mixed_precision.Policy('mixed_float16')
tf.keras.mixed_precision.set_global_policy(policy)
```

HuggingFace

```bash
pip install transformers datasets
```

```python
# Download model
model = AutoModel.from_pretrained("bert-base-uncased")

# Save locally
model.save_pretrained("./my_model")

# Load from local
model = AutoModel.from_pretrained("./my_model")

# Push to Hub
model.push_to_hub("username/model-name")
```

## 🔥 Hot Tips & Tricks

**1. Debugging**

- Check shapes: `print(tensor.shape)` everywhere
- NaN detection: `torch.isnan(tensor).any()`
- Gradient checking: `torch.autograd.gradcheck()`
- Set seed: `torch.manual_seed(42)` for reproducibility

**2. Performance**

- Use `DataLoader` with `num_workers=4+` and `pin_memory=True`
- Apply batch normalization before activation
- Use `torch.no_grad()` for inference
- Profile with `torch.profiler` or TensorBoard

**3. Memory**

- Use gradient accumulation for large effective batch sizes
- Delete unused tensors: `del tensor`
- Empty cache: `torch.cuda.empty_cache()`
- Use gradient checkpointing for very deep networks

## ⚠️ Common Pitfalls

- Forgetting `model.eval()` during inference
- Not detaching tensors from the computation graph
- Using the wrong loss function for the task
- Overfitting on small datasets (use regularization)
- Not shuffling training data
- Learning rate too high (divergence) or too low (slow convergence)

## 📊 Cheat Sheet Summary

**When to Use What**

- Tabular data: XGBoost, Random Forest
- Images: ResNet, EfficientNet
- Text: BERT, RoBERTa
- Generation: GPT, T5
- Time series: LSTM, Transformer
- Reinforcement learning: PPO, SAC, DQN

**Hyperparameters**

- Learning rate: `1e-4` to `3e-4` (Adam)
- Batch size: 32-256
- Dropout: 0.1-0.5
- Weight decay: `1e-5` to `1e-2`
- Warmup: 0-10% of total steps
- Gradient clip: 1.0

**Resources**

- Papers: arXiv.org
- Code: Papers with Code
- Models: HuggingFace Hub
- Data: Kaggle, UCI ML Repository
- Learn: Fast.ai, Coursera
- Community: Reddit r/MachineLearning

## 🎯 You've Got This!

This cheat sheet contains everything you need for quick reference. Bookmark it, keep it open in a tab, and use it often — keep building, keep learning, keep shipping.

---
*Source: adapted from the Generative AI cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
