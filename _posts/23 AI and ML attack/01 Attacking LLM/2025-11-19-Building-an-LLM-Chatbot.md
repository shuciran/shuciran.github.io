---
description: >-
  Building an LLM Chatbot
title: Building an LLM Chatbot     # Add title here
date: 2025-11-19 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, LLM Chatbot]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Requirements:
```bash
apt update
apt install python3-pip -y
cat >requirements.txt <<EOF
transformers==4.48.3
torch==2.6.0
accelerate==1.8.1
einops==0.8.1
jinja2==3.1.6
EOF
pip install -r requirements.txt
```
Python code:
```python
cat>>llm-chatbot.py<<EOF
from transformers import AutoModelForCausalLM, AutoTokenizer

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"

model = AutoModelForCausalLM.from_pretrained(
        "microsoft/Phi-3-mini-4k-instruct", #TinyLlama/TinyLlama-1.1B-Chat-v1.0
        revision=revision_id,
        device_map="auto",
        torch_dtype="auto",
        trust_remote_code=True,
        )

tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct", revision=revision_id)

from transformers import pipeline

generator = pipeline(
        "text-generation",
        model=model,
        tokenizer=tokenizer,
        return_full_text=False,
        max_new_tokens=500,
        do_sample=False
        )

while True:
    print("-" *50) # Horizontal line
    print("What do you want?")
    user_input = input("\033[92mType something, or X to exit: \033[0m") # Ask the user for input in green color
    if user_input in ['X', 'x']: # If user types X or x, exit the program
        print("Exiting.")
        break
    else:
        messages = [{"role":"user", "content": user_input}]
        response = generator(messages)
        print(response[0]["generated_text"])
EOF

python3 llm-chatbot.py
```
