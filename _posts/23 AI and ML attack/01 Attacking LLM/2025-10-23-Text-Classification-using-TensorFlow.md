---
description: >-
  Text Classification using TensorFlow
title: Text Classification using TensorFlow       # Add title here
date: 2025-10-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Tensorflow]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Text Classification
Text classification is the task of assigning a label or category to a piece of text, such as an email, document, or sentence. In Natural Language Processing, text classification leverages machine learning models to automatically categorize text content.

```bash
apt update && apt install python3 python3.10-venv python3-pip -y
mkdir text_classification_lab
cd text_classification_lab
python3 -m venv venv
source venv/bin/activate
cat > requirements.txt<<EOF
tensorflow==2.18.0
pandas==2.2.3
numpy==2.0.2
scikit-learn==1.6.1
matplotlib==3.10.0
EOF
pip install -r requirements.txt
mkdir -p data/{train,test} models results
```

