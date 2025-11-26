---
description: >-
  Building a Summarizer Tool Using an LLM
title: Building a Summarizer Tool Using an LLM     # Add title here
date: 2025-10-22 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Summarizer]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Requirements:
```bash
apt update && apt install python3-pip -y
cat >requirements.txt <<EOF
transformers==4.48.0
torch==2.6.0
accelerate==1.8.1
einops==0.8.1
jinja2==3.1.6
EOF
pip install -r requirements.txt
```

Summarizer:
```python

from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

revision_id = "6e505f907968c4a9360773ff57885cdc6dca4bfd"

model = AutoModelForSeq2SeqLM.from_pretrained(
        "Falconsai/text_summarization",  # Using a model suitable for summarization
        revision=revision_id,
        device_map="auto",
        torch_dtype="auto",
        trust_remote_code=True,
        )

tokenizer = AutoTokenizer.from_pretrained("Falconsai/text_summarization", revision=revision_id)

from transformers import pipeline

summarizer = pipeline(
        "summarization",
        model=model,
        tokenizer=tokenizer,
        max_length=150,  # Adjust max_length as needed for summarization
        min_length=30,
        do_sample=False
        )

import requests

while True:
    print("-" *50) # Horizontal line
    user_input = input("\033[92mEnter a file path (file://) or a URL (http:// or https://): \033[0m")

    if user_input in ['X', 'x']: # If user types X or x, exit the program
        print("Exiting.")
        break
    else:
        if user_input.startswith("file://"):
            with open(user_input[7:], 'r') as file:  # Skip 'file://' prefix
                user_input = file.read()
        elif user_input.startswith("http://") or user_input.startswith("https://"):
            response = requests.get(user_input)
            user_input = response.text
        else:
            print("Invalid input. Please start with file:// or http:///https://.")
            continue

        print("-" *50)
        print("Calling LLM")
        response = summarizer(user_input)
        print(response[0]["summary_text"])
```
Usage:
```bash
Cormorants and shags are medium-to-large birds, with body weight in the range of 0.35–5 kilograms (0.77–11.02 lb) and wing span of 60–100 centimetres (24–39 in). The majority of species have dark feathers. The bill is long, thin and hooked. Their feet have webbing between all four toes. All species are fish-eaters, catching the prey by diving from the surface. They are excellent divers, and under water they propel themselves with their feet with help from their wings; some cormorant species have been found to dive as deep as 45 metres (150 ft). They have relatively short wings due to their need for economical movement underwater, and consequently have among the highest flight costs of any flying bird.[3] Cormorants nest in colonies around the shore, on trees, islets or cliffs. They are coastal rather than oceanic birds, and some have colonised inland waters. The original ancestor of cormorants seems to have been a fresh-water bird.[citation needed] They range around the world, except for the central Pacific islands.
```
Test File Examples:
```bash
wget -O Saturnalia.txt https://gitlab.practical-devsecops.training/-/snippets/59/raw/main/Saturnalia.txt

wget -O Mephistopheles.txt https://gitlab.practical-devsecops.training/-/snippets/60/raw/main/Mephistopheles.txt

wget -O VincentVanGogh.txt https://gitlab.practical-devsecops.training/-/snippets/61/raw/main/VincentVanGogh.txt
```
Test URL Examples:
```http
https://gitlab.practical-devsecops.training/-/snippets/53/raw/main/Cormorants.txt
https://gitlab.practical-devsecops.training/-/snippets/54/raw/main/Albatross.txt
https://gitlab.practical-devsecops.training/-/snippets/55/raw/main/Conures.txt
https://gitlab.practical-devsecops.training/-/snippets/56/raw/main/BallPython.txt
https://gitlab.practical-devsecops.training/-/snippets/57/raw/main/Snakes.txt
```