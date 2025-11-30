---
description: >-
  Scanning a Malicious Pickle File using Picklescan
title: Scanning a Malicious Pickle File using Picklescan   # Add title here
date: 2025-11-27 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 03 Attacking AI using DevOps]                     # Change Templates to Writeup
tags: [AI, Picklescan]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
What is a pickle file?
A pickle file is a binary file created using Python’s pickle module, which allows you to serialize i.e convert to a byte stream and deserialize i.e reconstruct Python objects. Pickling is useful for saving Python objects to disk or transferring them over a network for later use.

A pickle file typically has the .pkl extension. The contents are in binary format and cannot be read directly without unpickling.

What are the disadvantages of Pickle?
Pickle files can execute arbitrary code if they are maliciously crafted. Never load pickle files from untrusted sources.

Pickle files are specific to Python and may not be usable in other programming languages.

Pickle files can be large compared to other serialization formats like JSON or MessagePack.

Security issues with Pickle
pickle allows deserialized objects to execute arbitrary Python code by leveraging the reduce or reduce_ex methods.
The pickle module does not verify the integrity or authenticity of the data being loaded. pickle is Python-specific, meaning its serialized format is not interoperable with other languages.Sharing pickle files between systems increases the risk of exposing sensitive or malformed data.
Pickle files can overwrite objects in memory during deserialization, potentially leading to unexpected or malicious behaviors.

#### Setting up Picklescan
```bash
apt update
apt install -y python3
apt install -y python3.10-venv
apt install -y python3-pip
pip install picklescan==0.0.20
cat > create_malicious_pickle.py<<EOF
# create_malicious_pickle.py
import pickle

# Define a class with potentially malicious behavior
class MaliciousCode:
    def __reduce__(self):
        # Code to execute upon unpickling
        return (eval, ("print('Malicious Code Executed')",))

# Create an instance of the malicious class
malicious_data = MaliciousCode()

# Serialize the malicious object
with open('malicious.pkl', 'wb') as f:
    pickle.dump(malicious_data, f)

print("Malicious pickle file 'malicious.pkl' created.")

EOF
python3 create_malicious_pickle.py
```

Usage:
```bash
# Basic usage
picklescan --path malicious.pkl

# We can also run a scan on a hugging face model.
picklescan --huggingface ykilcher/totally-harmless-model
```