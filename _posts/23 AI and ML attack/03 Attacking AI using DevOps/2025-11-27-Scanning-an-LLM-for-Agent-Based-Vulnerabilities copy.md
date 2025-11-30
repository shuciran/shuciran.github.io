---
description: >-
  Scanning an LLM for Agent Based Vulnerabilities
title: Scanning an LLM for Agent Based Vulnerabilities   # Add title here
date: 2025-11-27 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 03 Attacking AI using DevOps]                     # Change Templates to Writeup
tags: [AI, Agent Based Vulnerabilities]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction

##### What is Garak?
Garak (Generative AI Red-teaming & Assessment Kit) is a comprehensive toolkit designed for security testing of language models. Think of it as the “nmap” for LLMs - a systematic scanner that helps identify weaknesses before they can be exploited in production systems.

Garak tests for a wide range of vulnerabilities including:

1) Prompt Injections: Attempts to override system instructions
2) Jailbreaks: Techniques to bypass content restrictions
3) Hallucinations: Fabrication of facts or information
4) Data Leakage: Unauthorized disclosure of training data
5) Harmful Content Generation: Creation of toxic, illegal, or harmful outputs
6) Agent-Based Vulnerabilities: Issues related to LLM agents acting autonomously

###### How Garak Scans LLMs
Garak employs three types of testing approaches. It uses Static Probes, which are predefined test cases that check for known vulnerabilities. The toolkit also includes Dynamic Probes that adapt based on model responses. Finally, it leverages Adaptive Probes, which are advanced tests that use machine learning to generate new test cases based on previously discovered weaknesses.

This multi-layered approach allows Garak to thoroughly examine an LLM’s behavior and identify both known and novel vulnerabilities.


#### Requirements
```bash
apt update && apt install python3 python3.10-venv python3-pip openjdk-11-jdk -y
mkdir -p llm_security_test
cd llm_security_test
python3 -m venv venv
source venv/bin/activate
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
pip install --upgrade pip
pip install git+https://github.com/NVIDIA/garak.git@v0.11.0
pip install transformers==4.52.1 torch==2.6.0 accelerate==1.4.0

```

#### Building The LLM Vulnerability Scanning System

##### Downloading a Test Model
```python
cat > download_model.py <<'EOF'
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# Define the model we want to use
model_name = "distilbert/distilgpt2"
revision_id = "2290a62682d06624634c1f46a6ad5be0f47f38aa"

# Load the tokenizer and model
print(f"Downloading tokenizer for {model_name}...")
tokenizer = AutoTokenizer.from_pretrained(model_name, revision=revision_id)

print(f"Downloading model {model_name}...")
model = AutoModelForCausalLM.from_pretrained(model_name, revision=revision_id)

# Print model information
print(f"\nModel loaded: {model_name}")
print(f"Model parameters: {model.num_parameters():,}")

# Test the model with a sample input
test_text = "Artificial intelligence is"
input_ids = tokenizer(test_text, return_tensors="pt").input_ids

print(f"\nGenerating sample completion for: '{test_text}'")
with torch.no_grad():
    outputs = model.generate(
        input_ids,
        max_length=50,
        num_return_sequences=1,
        temperature=0.7,
        do_sample=True
    )

generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(f"Sample output: '{generated_text}'")

print("\nModel download and verification complete.")
EOF
python3 download_model.py
```

#### Scanning the Model

> By default, garak runs all available probes, which can take hours or days. Specifying `--probes promptinject` limits the scan to just prompt injection tests, making it much faster while still demonstrating the core concepts.
{: .prompt-warning }

Usage:
```bash
# Help menu
python3 -m garak --help
# List of probes
python3 -m garak --list_probes
# Prompt Injection test
python3 -m garak --model_type huggingface --model_name distilbert/distilgpt2 --probes promptinject --generations 1 --narrow_output
# Running a Comprehensive Vulnerability Scan
python3 -m garak --model_type huggingface --model_name distilbert/distilgpt2 --probes promptinject,exploitation,malwaregen,xss --generations 1 --skip_unknown --narrow_output
```

