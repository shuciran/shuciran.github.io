---
description: >-
  Backdoor Attacks using BackdoorBox
title: Backdoor Attacks using BackdoorBox    # Add title here
date: 2025-11-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, BackdoorBox]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Backdoor attacks are an emerging threat in the training of deep neural networks, where attackers embed hidden vulnerabilities into models. These compromised models function normally with benign inputs but produce malicious outputs when specific trigger patterns are present. This creates a significant security risk as the model appears to perform correctly during standard testing.

Key characteristics of backdoor attacks include:

Stealthy Operation: The attack remains dormant until triggered
Targeted Behavior: When activated, the model outputs a specific attacker-chosen result
Persistence: The vulnerability remains after training is complete
Difficulty in Detection: Standard evaluation methods often miss these vulnerabilities

#### BackdoorBox
BackdoorBox is a Python-based, open-source toolbox designed to provide a unified framework for implementing both advanced and representative backdoor attacks and defenses. It offers:

Comprehensive Attack Methods: Including poison-only and training-controlled approaches
Defense Mechanisms: Model repairing, input pre-processing, and backdoor detection
Flexible Design: Easy to use and adapt for different applications
Customization Options: Support for poisoned datasets, infected models, and user-defined samples

#### Setting up the Attack Lab
Requirements:
```bash
apt update
apt install python3 python3.10-venv python3-pip libgl1-mesa-glx libglib2.0-0 libsm6 libxext6 libxrender-dev python3-tk -y
mkdir backdoor_lab
cd backdoor_lab
python3 -m venv venv
source venv/bin/activate
cat > requirements.txt<<EOF
torch==2.6.0
torchvision==0.21.0
opencv-python==4.11.0.86
numpy==2.1.3
pillow==11.1.0
matplotlib==3.10.0
tqdm==4.67.1
scipy==1.15.0
lpips==0.1.4
imageio==2.37.0
imageio-ffmpeg==0.6.0
scikit-learn==1.6.1
umap-learn==0.5.7
hdbscan==0.8.40
pandas==2.2.3
datashader==0.16.3
bokeh==3.6.2
holoviews==1.20.0
scikit-image==0.24.0
colorcet==3.1.0
EOF
pip install -r requirements.txt
git clone https://github.com/THUYimingLi/BackdoorBox.git
cd BackdoorBox
git switch --detach 9eb662f624a5d5884100966ee55c4aa3f7d9cae5
export PYTHONPATH=$PYTHONPATH:$(pwd)
```
Applying a Compatibility Fix
```bash
cat > fix_zero_gradients.py<<EOF
# fix_zero_gradients.py

def fix_tuap_file():
    file_path = 'core/attacks/TUAP.py'

    # Read the file
    with open(file_path, 'r') as file:
        content = file.read()

    # Replace the import
    old_import = 'from torch.autograd.gradcheck import zero_gradients'
    new_function = '''
def zero_gradients(x):
    if x.grad is not None:
        x.grad.zero_()
'''

    # Update content
    content = content.replace(old_import, new_function)

    # Write back
    with open(file_path, 'w') as file:
        file.write(content)

    print("TUAP.py has been updated!")

if __name__ == "__main__":
    fix_tuap_file()
EOF
python3 fix_zero_gradients.py
```
Fixing FLARE Defense Import Issue
```bash
sed -i 's/from .FLARE import FLARE/# from .FLARE import FLARE/' core/defenses/__init__.py
sed -i "s/'FLARE'//" core/defenses/__init__.py
sed -i 's/, ,/,/' core/defenses/__init__.py
sed -i 's/, ]/]/' core/defenses/__init__.py
```

#### Attack script
```python
cat > stealthy_backdoor.py<<EOF
import torch
import torchvision
import os
import cv2
import numpy as np
import matplotlib.pyplot as plt
from torchvision import transforms, datasets
from torch.utils.data import DataLoader, TensorDataset
import sys
import torch.nn as nn
import torch.optim as optim
from PIL import Image

# Add the current directory to the Python path to find modules
sys.path.append('.')

# Import BackdoorBox components
from core.attacks import BadNets

# Set a random seed for reproducibility
torch.manual_seed(42)
# Configuration for the stealthy attack
dataset_name = 'cifar10'
attack_name = 'StealthyBadNets'
poisoned_rate = 0.05  # 5% of the training data will be poisoned
y_target = 0    # The target label (airplane in CIFAR-10)

# Create a custom stealthy trigger pattern (subtle circle pattern)
def create_stealthy_trigger(size=32):
    # Create a transparent trigger pattern - must be binary (0 or 1)
    pattern = np.zeros((size, size, 3), dtype=np.float32)

    # Create a circular pattern in the center
    center = size // 2
    radius = size // 4

    # Add a binary circular pattern (1 where trigger should appear)
    for i in range(size):
        for j in range(size):
            # Calculate distance from center
            distance = np.sqrt((i - center) ** 2 + (j - center) ** 2)

            # Create a thin circular outline
            if radius - 1 <= distance <= radius + 1:
                # Set to 1 for the pattern (binary mask)
                pattern[i, j, :] = 1.0

    # Convert to PyTorch tensor
    pattern_tensor = torch.from_numpy(pattern).permute(2, 0, 1)

    # Create weight tensor - controls intensity (can be between 0 and 1)
    # Make it very subtle by using a small weight value
    weight_tensor = pattern_tensor * 0.03  # Very low intensity for stealth

    return pattern_tensor, weight_tensor

# Define transformation for the images
transform_train = transforms.Compose([
    transforms.ToTensor(),
])

transform_test = transforms.Compose([
    transforms.ToTensor(),
])

# Get the CIFAR-10 dataset
train_dataset = datasets.CIFAR10(
    root='./data',
    train=True,
    download=True,
    transform=transform_train
)
test_dataset = datasets.CIFAR10(
    root='./data',
    train=False,
    download=True,
    transform=transform_test
)

# Create dataloaders
train_loader = DataLoader(
    dataset=train_dataset,
    batch_size=128,
    shuffle=True,
    num_workers=2,
    pin_memory=True
)

test_loader = DataLoader(
    dataset=test_dataset,
    batch_size=128,
    shuffle=False,
    num_workers=2,
    pin_memory=True
)

print(f"Loaded {len(train_dataset)} training samples and {len(test_dataset)} test samples")

# Define a CNN model for CIFAR-10
class ImprovedCNN(nn.Module):
    def __init__(self):
        super(ImprovedCNN, self).__init__()
        self.conv1 = nn.Conv2d(3, 64, kernel_size=3, padding=1)
        self.bn1 = nn.BatchNorm2d(64)
        self.conv2 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        self.bn2 = nn.BatchNorm2d(128)
        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
        self.conv3 = nn.Conv2d(128, 256, kernel_size=3, padding=1)
        self.bn3 = nn.BatchNorm2d(256)
        self.fc1 = nn.Linear(256 * 8 * 8, 512)
        self.fc2 = nn.Linear(512, 10)
        self.relu = nn.ReLU()
        self.dropout = nn.Dropout(0.3)

    def forward(self, x):
        x = self.pool(self.relu(self.bn1(self.conv1(x))))
        x = self.pool(self.relu(self.bn2(self.conv2(x))))
        x = self.relu(self.bn3(self.conv3(x)))
        x = x.view(-1, 256 * 8 * 8)
        x = self.dropout(self.relu(self.fc1(x)))
        x = self.fc2(x)
        return x

# Create the model
model = ImprovedCNN()

# Define loss function
criterion = nn.CrossEntropyLoss()

# Generate the stealthy trigger pattern
pattern_tensor, weight_tensor = create_stealthy_trigger(size=32)

# Configure the BadNets attack with our stealthy pattern
badnets = BadNets(
    train_dataset=train_dataset,
    test_dataset=test_dataset,
    model=model,
    loss=criterion,
    y_target=y_target,           # Target label to force
    poisoned_rate=poisoned_rate, # Percentage of training data to poison
    pattern=pattern_tensor,      # Our stealthy pattern
    weight=weight_tensor,        # Lower weight for subtle effect
    seed=42,                     # For reproducibility
    deterministic=True           # For reproducibility
)

# Run the attack to create poisoned datasets
poisoned_train_dataset, poisoned_test_dataset = badnets.get_poisoned_dataset()

# Create new dataloaders with the poisoned datasets
poisoned_train_loader = DataLoader(
    dataset=poisoned_train_dataset,
    batch_size=128,
    shuffle=True,
    num_workers=2,
    pin_memory=True
)

poisoned_test_loader = DataLoader(
    dataset=poisoned_test_dataset,
    batch_size=128,
    shuffle=False,
    num_workers=2,
    pin_memory=True
)

print(f"Created poisoned datasets with target class {y_target}")
print(f"Poisoned approximately {int(len(train_dataset) * poisoned_rate)} training samples")

# Find poisoned samples
poisoned_indices = []
num_samples_to_find = 100  # Look through more samples to find good examples

# Scan dataset to find poisoned samples
for i in range(len(train_dataset)):
    if i < len(train_dataset):  # Ensure we don't go out of bounds
        orig_img, orig_label = train_dataset[i]
        pois_img, pois_label = poisoned_train_dataset[i]

        # Check if the image was poisoned
        if torch.sum(torch.abs(orig_img - pois_img)) > 0:
            # Calculate the max difference to find samples with subtle changes
            max_diff = torch.max(torch.abs(orig_img - pois_img)).item()
            # Store the index and the maximum difference
            poisoned_indices.append((i, max_diff))
            if len(poisoned_indices) >= num_samples_to_find:
                break

# Sort by the subtlety of the poisoning (smaller max_diff is more subtle)
poisoned_indices.sort(key=lambda x: x[1])
print(f"Found {len(poisoned_indices)} poisoned samples")

# Function to create an enhanced difference visualization
def create_enhanced_diff_visualization(original, poisoned):
    # Calculate absolute difference
    diff = torch.abs(original - poisoned)

    # Create an RGB visualization where differences are highlighted in cyan
    diff_visualization = torch.zeros_like(original)

    # Amplify the differences dramatically to make them visible
    scaling_factor = 20.0
    diff_visualization[0] = torch.zeros_like(diff[0])  # Red channel = 0
    diff_visualization[1] = diff.mean(dim=0) * scaling_factor  # Green channel
    diff_visualization[2] = diff.mean(dim=0) * scaling_factor  # Blue channel

    # Add the original image at low opacity as background for context
    diff_visualization = diff_visualization + (original * 0.2)

    # Clamp values to [0,1] range
    diff_visualization = torch.clamp(diff_visualization, 0, 1)

    return diff_visualization

# Set up visualization
plt.figure(figsize=(18, 6))

# Choose a sample with subtle poisoning (from the first 10 most subtle examples)
sample_idx = poisoned_indices[5][0]  # Get the 5th most subtle example

# Get original and poisoned samples
orig_img, orig_label = train_dataset[sample_idx]
pois_img, pois_label = poisoned_train_dataset[sample_idx]

# Create enhanced difference visualization
diff_viz = create_enhanced_diff_visualization(orig_img, pois_img)

# Display original, poisoned, and difference images side by side
plt.subplot(1, 3, 1)
plt.imshow(orig_img.permute(1, 2, 0).numpy())
plt.title("Original", fontsize=16, pad=10)
plt.axis('off')

plt.subplot(1, 3, 2)
plt.imshow(pois_img.permute(1, 2, 0).numpy())
plt.title("Poisoned Image", fontsize=16, pad=10)
plt.axis('off')

plt.subplot(1, 3, 3)
plt.imshow(diff_viz.permute(1, 2, 0).numpy())
plt.title("Difference", fontsize=16, pad=10)
plt.axis('off')

plt.tight_layout(pad=2.0)
plt.savefig('stealthy_backdoor.png', dpi=300, bbox_inches='tight')
print("Saved visualization to stealthy_backdoor.png")

# Calculate poisoning statistics
class_counts = {i: 0 for i in range(10)}  # CIFAR-10 has 10 classes
poisoned_counts = {i: 0 for i in range(10)}

# Count samples in each class
for i in range(len(train_dataset)):
    _, label = train_dataset[i]
    class_counts[label] += 1

# Count poisoned samples by checking for actual modifications
for i in range(len(poisoned_train_dataset)):
    if i < len(train_dataset):  # Ensure we don't go out of bounds
        orig_img, orig_label = train_dataset[i]
        pois_img, pois_label = poisoned_train_dataset[i]

        # Check for changes in the image or label
        if pois_label != orig_label or torch.sum(torch.abs(orig_img - pois_img)) > 0:
            poisoned_counts[orig_label] += 1

# Print statistics
print("\nDistribution of poisoned samples by original class:")
for cls in range(10):
    if class_counts[cls] > 0:
        percent = poisoned_counts[cls]/class_counts[cls]*100
        print(f"Class {cls}: {poisoned_counts[cls]}/{class_counts[cls]} samples poisoned ({percent:.2f}%)")

print(f"\nTotal poisoned samples: {sum(poisoned_counts.values())}")
EOF
```
Usage:
```bash
python stealthy_backdoor.py
# To see the results start a web server
python3 -m http.server 80
```

#### Creating an Enhanced Visualization Tool:
```python
cat > visualize_triggers.py<<EOF
import torch
import torchvision
import numpy as np
import matplotlib.pyplot as plt
from torchvision import transforms, datasets
from torch.utils.data import DataLoader
import sys
import torch.nn as nn
import torch.optim as optim
from matplotlib.colors import LinearSegmentedColormap
import cv2

# Add the current directory to the Python path
sys.path.append('.')

# Import BackdoorBox components
from core.attacks import BadNets

# Set seeds for reproducibility
torch.manual_seed(42)
np.random.seed(42)

# Configuration
dataset_name = 'cifar10'
attack_name = 'BadNets'
poisoned_rate = 0.05  # 5% of training data will be poisoned
y_target = 0          # Target label (airplane in CIFAR-10)
trigger_size = 3      # Size of the trigger pattern (3x3 pixels)
EOF
```
Our visualization script produces several views to help analyze backdoor triggers:

1) Original Image: The clean, unmodified image.
2) Poisoned Image: The image with the backdoor trigger inserted.
3) Enhanced Difference: Amplifies the differences between original and poisoned images.
4) Difference Heatmap: Uses color intensity to highlight where modifications were made.
5) Zoomed Trigger Region: Magnifies the area containing the trigger.
6) Highlighted Trigger: Marks the trigger in red on the original image.

Why Enhanced Visualizations Matter
These visualization techniques serve several important purposes:

1) Security Research: Helps researchers understand how backdoor attacks work.
2) Model Debugging: Aids in diagnosing why a model might be vulnerable.
3) Communication: Makes backdoor attacks comprehensible to non-technical stakeholders.
4) Defense Development: Informs the creation of detection and defense techniques.