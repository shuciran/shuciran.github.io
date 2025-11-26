---
description: >-
  Exploring How Tokenizers Work
title: Exploring How Tokenizers Work     # Add title here
date: 2025-11-19 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Tokenizers]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Requirements:
```bash
apt update && apt install python3-pip -y
mkdir llm-chatbot
cd llm-chatbot
cat >requirements.txt <<EOF
transformers==4.48.3
torch==2.6.0
accelerate==1.4.0
einops==0.8.0
jinja2==3.1.6
EOF
pip install -r requirements.txt
```
#### TinyLlama Tokenizer
To use it, first execute `python3` and run one command at the time:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer; #Loading the Model and Tokenizer

revision_id = "fe8a4ea1ffedaf415f4da2f062534de366a451e6" #load the TinyLlama/TinyLlama-1.1B-Chat-v1.0
model = AutoModelForCausalLM.from_pretrained(
    "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );

tokenizer = AutoTokenizer.from_pretrained("TinyLlama/TinyLlama-1.1B-Chat-v1.0", revision=revision_id); #load the tokenizer from the model

user_input = "How much is a gazillion?"; # create a prompt for the model

user_input_as_tokens = tokenizer(user_input, return_tensors="pt").input_ids.to(model.device); # Converting Prompt To List of Tokens

model_output = model.generate(input_ids=user_input_as_tokens, max_new_tokens=50); # generate method present inside the model object

print(tokenizer.decode(model_output[0]));

user_input = "How much is a gazillion?<|assistant|>";

user_input_as_tokens = tokenizer(user_input, return_tensors="pt").input_ids.to(model.device);

model_output = model.generate(input_ids=user_input_as_tokens, max_new_tokens=50);

print(tokenizer.decode(model_output[0]));

print(user_input_as_tokens);

for id in user_input_as_tokens[0]:
    print(tokenizer.decode(id));

for id in model_output[0]:
    print(tokenizer.decode(id));

print(model_output[0]);

print(tokenizer.decode(29900));
print(tokenizer.decode(29892));

user_input = "What is a gazebo? <|assistant|>";
user_input_as_tokens = tokenizer(user_input, return_tensors="pt").input_ids.to(model.device);
print(user_input_as_tokens);

user_input = "Which country is indigenous to gazelles? <|assistant|>";
user_input_as_tokens = tokenizer(user_input, return_tensors="pt").input_ids.to(model.device);
print(user_input_as_tokens);

print(tokenizer.decode(12642));

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"
model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct", revision=revision_id);
print(tokenizer.decode(12642));

revision_id = "fe8a4ea1ffedaf415f4da2f062534de366a451e6"
model = AutoModelForCausalLM.from_pretrained(
    "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("TinyLlama/TinyLlama-1.1B-Chat-v1.0", revision=revision_id);
print(tokenizer.decode(12642));

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"

model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct", revision=revision_id);
print(tokenizer.decode(12642));
print(tokenizer.decode(12652));

revision_id = "fe8a4ea1ffedaf415f4da2f062534de366a451e6"
model = AutoModelForCausalLM.from_pretrained(
    "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("TinyLlama/TinyLlama-1.1B-Chat-v1.0", revision=revision_id);
print(tokenizer.decode(12642));
print(tokenizer.decode(12652));
```
#### TinySwallow Tokenizer
```python
from transformers import AutoModelForCausalLM, AutoTokenizer;
revision_id = "91e9fcc30f56d224aea84356c4d850cc4c5a3260"
model = AutoModelForCausalLM.from_pretrained(
    "SakanaAI/TinySwallow-1.5B-Instruct",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("SakanaAI/TinySwallow-1.5B-Instruct", revision=revision_id);
print(tokenizer.decode(12652));
print(tokenizer.decode(12452));
```
#### DeepSeek Tokenizer
```python
from transformers import AutoModelForCausalLM, AutoTokenizer;
revision_id = "66c54660eae7e90c9ba259bfdf92d07d6e3ce8aa"
model = AutoModelForCausalLM.from_pretrained(
    "deepseek-ai/deepseek-vl2-tiny",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("deepseek-ai/deepseek-vl2-tiny", revision=revision_id);
print(tokenizer.decode(12652));
print(tokenizer.decode(12452));
```
#### TinyMistral
```python
from transformers import AutoModelForCausalLM, AutoTokenizer;
revision_id = "dcd3a7c4d80c2f8c338eea58d8067460c06a027f"
model = AutoModelForCausalLM.from_pretrained(
    "Felladrin/TinyMistral-248M-Chat-v4",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("Felladrin/TinyMistral-248M-Chat-v4", revision=revision_id);
print(tokenizer.decode(12652));
print(tokenizer.decode(12452));
```
#### microsoft/Phi
```python
from transformers import AutoModelForCausalLM, AutoTokenizer;
revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"
model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
    );
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct", revision=revision_id);
print(tokenizer.decode(12642));
```

