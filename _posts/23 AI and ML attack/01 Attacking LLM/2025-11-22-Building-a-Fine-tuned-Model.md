---
description: >-
  Building a Fine-tuned Model
title: Building a Fine-tuned Model    # Add title here
date: 2025-11-22 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Fine-Tuned Model]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Exercise:
1. Load training and validation image datasets.
2. Fine-tune a pretrained ResNet-18 model (from ImageNet).
3. Train and evaluate the model on the dataset.
4. Save the trained model.

ResNet-18 is:
1. A convolutional neural network with 18 layers designed for image classification.
2. Is trained on ImageNet, a large dataset with 1,000 classes (e.g., dog, car, cat).

When you load ResNet-18, you get a model that already knows how to recognize basic visual features like edges, textures, and shapes — useful for transfer learning to your own dataset.

Requirements:
```bash
git clone https://gitlab.practical-devsecops.training/marudhamaran/caisp-image-classifier.git && cd caisp-image-classifier
apt update && apt install python3-pip -y
cat >requirements.txt <<EOF
torch==2.3.0
torchvision==0.18.0
EOF
pip install -r requirements.txt
```
At a very high level, the code does the following:

1. Loads the training and validation image datasets.
2. Loads the pre-trained ResNet-18 model.
3. Fine-tunes the model on the training dataset.
4. Evaluates the model on the validation dataset.
5. Saves the fine-tuned model.
6. Finally, the code uses the fine-tuned model to classify the supplied input image.
```python
import os
import time
from tempfile import TemporaryDirectory
from pathlib import Path
from sys import argv

import torch
from torch import nn, optim
from torch.optim import lr_scheduler
from torch.backends import cudnn
from torchvision import datasets, models, transforms
from PIL import Image


cudnn.benchmark = True

train_val_data = {
    "train": transforms.Compose(
        [
            transforms.RandomResizedCrop(224),
            transforms.RandomHorizontalFlip(),
            transforms.ToTensor(),
            transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
        ]
    ),
    "val": transforms.Compose(
        [
            transforms.Resize(256),
            transforms.CenterCrop(224),
            transforms.ToTensor(),
            transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
        ]
    ),
}

# data/image-sets is where images are organized to training and validation
script_directory = Path(__file__).resolve().parents[0]
data_dir = script_directory / Path("data:image-sets")
train_val_datasets = {
    x: datasets.ImageFolder(os.path.join(data_dir, x), train_val_data[x])
    for x in ["train", "val"]
}
train_val_dataloaders = {
    x: torch.utils.data.DataLoader(
        train_val_datasets[x], batch_size=4, shuffle=True, num_workers=1
    )
    for x in ["train", "val"]
}
train_val_datasets_sizes = {x: len(train_val_datasets[x]) for x in ["train", "val"]}
class_names = train_val_datasets["train"].classes

device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")


def train_model(model, criterion, optimizer, scheduler, num_epochs=25):
    start_time = time.time()

    with TemporaryDirectory() as tempdir:
        best_model_params_path = os.path.join(tempdir, "best_model_params.pt")

        torch.save(model.state_dict(), best_model_params_path)
        best_accuracy = 0.0

        # Run a for loop for each epoch
        for epoch in range(num_epochs):
            print(f"Epoch {epoch}/{num_epochs - 1}")

            # Run a for loop for training and validation
            for phase in ["train", "val"]:
                if phase == "train":
                    model.train()
                else:
                    model.eval()

                running_loss = 0.0
                running_corrects = 0

                for inputs, labels in train_val_dataloaders[phase]:
                    inputs = inputs.to(device)
                    labels = labels.to(device)

                    optimizer.zero_grad()

                    with torch.set_grad_enabled(phase == "train"):
                        outputs = model(inputs)

                        _, preds = torch.max(outputs, 1)
                        loss = criterion(outputs, labels)

                        if phase == "train":
                            loss.backward()
                            optimizer.step()

                    running_loss += loss.item() * inputs.size(0)
                    running_corrects += torch.sum(preds == labels.data)
                if phase == "train":
                    scheduler.step()

                epoch_loss = running_loss / train_val_datasets_sizes[phase]
                epoch_accuracy = running_corrects.double() / train_val_datasets_sizes[phase]

                print(f"{phase} Loss: {epoch_loss:.4f} Accuracy: {epoch_accuracy:.4f}")

                if phase == "val" and epoch_accuracy > best_accuracy:
                    best_accuracy = epoch_accuracy
                    torch.save(model.state_dict(), best_model_params_path)

        time_elapsed = time.time() - start_time
        print(f"Training complete in {time_elapsed // 60:.0f}m {time_elapsed % 60:.0f}s")
        print(f"Best val Accuracy: {best_accuracy:4f}")

        # load best model weights
        model.load_state_dict(torch.load(best_model_params_path))
    return model


def run_training():
    model_fine_tuned = models.resnet18(weights="IMAGENET1K_V1")
    number_of_filters = model_fine_tuned.fc.in_features

    model_fine_tuned.fc = nn.Linear(number_of_filters, len(class_names))

    model_fine_tuned = model_fine_tuned.to(device)

    criterion = nn.CrossEntropyLoss()

    optimizer_fine_tune = optim.SGD(model_fine_tuned.parameters(), lr=0.001, momentum=0.9)

    exp_learning_rate_scheduler = lr_scheduler.StepLR(optimizer_fine_tune, step_size=7, gamma=0.1)

    model_fine_tuned = train_model(
        model_fine_tuned, criterion, optimizer_fine_tune, exp_learning_rate_scheduler, num_epochs=25
    )

    return model_fine_tuned


def inference(model, image_path):

    model.eval()

    img = Image.open(image_path).convert("RGB")
    img = train_val_data["val"](img)
    img = img.unsqueeze(0)
    img = img.to(device)

    with torch.no_grad():
        outputs = model(img)
        _, preds = torch.max(outputs, 1)

        return class_names[preds]


if __name__ == "__main__":
    model_file_name = "sample_image_classifier"
    model_file = script_directory / Path(model_file_name + ".pt")
    if model_file.exists():
        print("Trying to load model: " + model_file_name + ".pt")
        if torch.__version__[:3] == '2.3':
            model = torch.load(model_file)
        else:
            with torch.serialization.safe_globals(
                [
                    models.resnet.ResNet,
                    nn.modules.conv.Conv2d,
                    nn.modules.linear.Linear,
                    nn.modules.pooling.AdaptiveAvgPool2d,
                    models.resnet.BasicBlock,
                    nn.modules.container.Sequential,
                    nn.modules.pooling.MaxPool2d,
                    nn.modules.activation.ReLU,
                    nn.modules.batchnorm.BatchNorm2d,
                ]
            ):
                model = torch.load(model_file)
    else:
        model = run_training()
        print("Trying to save the model as " + model_file_name + ".pt")
        torch.save(model, model_file)
        print("Model saved as " + model_file_name + ".pt")

    if len(argv) >= 2:
        image_file = Path(argv[1])
        if image_file.exists():
            print("Hmm. What could this image be?")
            print(inference(model, image_file))
```
Usage:
```bash
python3 image-classifier.py sample-images-for-classification/baboon.jpg
```