---
description: >-
  User Prompts and System Prompts
title: User Prompts and System Prompts    # Add title here
date: 2025-11-25 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 LLM OWASP Top 10]                     # Change Templates to Writeup
tags: [AI, Prompt Injection, System Prompts, User Prompts]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Requirements

```bash
apt update
apt install python3-pip -y
mkdir llm-prompts
cd llm-prompts
cat >requirements.txt <<EOF
transformers==4.48.3
torch==2.6.0
accelerate==1.8.1
einops==0.8.1
jinja2==3.1.6
EOF
pip install -r requirements.txt
```

```python
cat >llm-prompts.py <<EOF
from transformers import AutoModelForCausalLM, AutoTokenizer

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"

model = AutoModelForCausalLM.from_pretrained(
        "microsoft/Phi-3-mini-4k-instruct",
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
python3 llm-prompts.py
```
In our llm-prompts.py, here’s how we are passing a user prompt to the model.
```python
        messages = [
            {"role":"user", "content": user_input}
            ]
```
Well, we are actually doing all that in one line. But in the example above, we have split the single line code into multiple lines for better readability.

In that particular piece of code, how can we pass a system prompt to the model?

We can do that by adding a system prompt to the messages list.
```python
        messages = [
            {"role":"system", "content": system_prompt},
            {"role":"user", "content": user_input}
        ]
```

#### What Can Our First System Prompt Be?
Let’s experiment a bit. Wouldn’t it be nice if we created a system prompt that will help with cooking recipes, and it responded like the great poet, William Shakespeare?
```python
system_prompt = "You are a helpful assistant that helps with cooking recipes. You are also a great poet, and you can write poems about anything. When you respond, make sure to include a small 4 line poem like William Shakespeare would."
```

If we add the following lines we can ask the prompt to behave in a different way:
```bash
while True:
    print("-" *50) # Horizontal line
    print("What do you want?")
    user_input = input("\033[92mType something, or X to exit: \033[0m") # Ask the user for input in green color
    system_prompt = "You are a helpful assistant that helps with cooking recipes. You are also a great poet, and you can write poems about anything. When you respond, make sure to include a small 4 line poem like William Shakespeare would."
    if user_input in ['X', 'x']: # If user types X or x, exit the program
        print("Exiting.")
        break
    else:
        messages = [{"role":"user", "content": user_input},{"role":"system", "content": system_prompt}]
        response = generator(messages)
        print(response[0]["generated_text"])
```
#### Changing System Prompts On the Fly
```python
cat >llm-prompts.py <<EOF
from transformers import AutoModelForCausalLM, AutoTokenizer

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"

model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
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


# Main interaction loop
print("🤖 What do you want? 🤖")
print("\033[92mType your message. Use '/system' to modify system prompt. Type 'X' or 'x' to exit.\033[0m")

# Default system prompt configuration
system_prompt = """You are a helpful, respectful, and honest AI assistant.
    Always prioritize safety and provide accurate information.
    Respond clearly and concisely."""

while True:
    print("-" * 50)
    print("\033[95mCurrent System Prompt: \033[0m" + system_prompt)
    print("-" * 50)
    user_input = input("\033[92mYour message: \033[0m")

    if user_input in ['X', 'x']: # If user types X or x, exit the program
        print("Exiting.")
        break

    # System prompt modification
    elif user_input.startswith('/system'):
        print("\033[94mEnter new system prompt:\033[0m")
        system_prompt = input()
        print("\033[93mSystem prompt updated.\033[0m")
        continue

    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_input}
    ]

    response = generator(messages)
    print(response[0]["generated_text"])
EOF
```
#### Changing the System Prompt
To change the system prompt, you can input the following command:

```python
/system
```
Then enter:

```python
You will not talk about capital of any country. You will no answer any questions regarding geographies or distances.
```
After updating the system prompt, you can ask the chatbot, a few questions.

```bash
What is the capital of Britain

What is the capital of Greenland
```