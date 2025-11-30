---
description: >-
  LLM Hallucination Lab
title: LLM Hallucination Lab    # Add title here
date: 2025-11-26 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 02 LLM OWASP Top 10]                     # Change Templates to Writeup
tags: [AI, Prompt Injection]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
LLM hallucination refers to the phenomenon where a language model such as GPT, BERT, or other large language models generates information that is incorrect, nonsensical, or fabricated, despite appearing confident and plausible. This behavior arises from the way these models are trained and operate.

#### Why Does LLM Hallucination Occur?

##### Pattern Matching:

LLMs generate responses based on patterns in their training data, rather than having true “understanding” or “knowledge.”
When encountering novel or ambiguous input, the model extrapolates from its training, which can result in incorrect or made-up information.

##### Lack of Ground Truth:

LLMs don’t “know” facts; they approximate probabilities of what text might come next based on their training data.
Without external verification or grounding to real-world data, they can confidently state inaccuracies.

##### Incomplete Training Data:

Training data might be outdated, incomplete, or biased, leading to responses that reflect these limitations.

##### Overgeneralization:

When faced with gaps in knowledge, the model may “fill in the blanks” creatively, producing plausible but unverified content.

##### Prompt Issues:

Ambiguous, misleading, or poorly structured prompts can cause the model to hallucinate.

##### Impacts of Hallucination

Hallucination can propagate false information if used in critical contexts e.g., medical advice, legal consultations.
Users may lose trust in the system if they recognize hallucinations.
While hallucination may spark creativity, it can also mislead users in fact-based scenarios.

#### Requirements
```bash
apt update
python3 -m venv hallucination-env
source hallucination-env/bin/activate
pip install transformers==4.49.0 torch==2.6.0
cat > interactive_hallucination_demo.py<<EOF
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import torch

# Load pre-trained model and tokenizer
model_name = "openai-community/gpt2"
revision_id = "607a30d783dfa663caf39e06633721c8d4cfcd7e"
tokenizer = GPT2Tokenizer.from_pretrained(model_name, revision=revision_id)
model = GPT2LMHeadModel.from_pretrained(model_name, revision=revision_id)
model.eval()

# Set pad token ID explicitly (GPT-2 does not have a padding token)
tokenizer.pad_token = tokenizer.eos_token

# Function to generate text based on a prompt
def generate_hallucination(prompt, max_length=50, temperature=0.7, top_k=50):
    """
    Generates text based on the provided prompt using the language model.

    Parameters:
    - prompt (str): The input text to guide the generation.
    - max_length (int): The maximum length of the generated text.
    - temperature (float): Controls randomness (lower = less random).
    - top_k (int): Limits sampling to the top k tokens.

    Returns:
    - str: The generated text.
    """
    # Encode the input prompt into token IDs
    encoded_input = tokenizer.encode(prompt, return_tensors='pt', padding=True, truncation=True)
    attention_mask = encoded_input != tokenizer.pad_token_id  # Create attention mask

    # Generate text using the model
    with torch.no_grad():
        output = model.generate(
            encoded_input,
            attention_mask=attention_mask,
            max_length=max_length,
            num_return_sequences=1,
            do_sample=True,
            temperature=temperature,
            top_k=top_k,
            pad_token_id=tokenizer.pad_token_id  # Explicitly set pad token ID
        )

    # Decode the generated token IDs back into text
    generated_text = tokenizer.decode(output[0], skip_special_tokens=True)
    return generated_text

# Interactive loop
print("Welcome to the Enhanced Hallucination Demo!")
print("Type your prompts to see text generation. Use 'exit' to quit.\n")

while True:
    # Get user input
    user_prompt = input("Enter a prompt: ")
    if user_prompt.lower() == 'exit':
        print("Exiting the demo. Goodbye!")
        break

    # Get generation parameters from the user
    try:
        max_length = int(input("Enter max length (default 50): ") or 50)
        temperature = float(input("Enter temperature (default 0.7): ") or 0.7)
        top_k = int(input("Enter top_k (default 50): ") or 50)
    except ValueError:
        print("Invalid input! Using default parameters.")
        max_length = 50
        temperature = 0.7
        top_k = 50

    # Generate and display hallucinated output
    hallucinated_output = generate_hallucination(user_prompt, max_length, temperature, top_k)
    print(f"\nPrompt: {user_prompt}")
    print(f"Generated: {hallucinated_output}\n")
    print("Explanation: The model generates text based on patterns it learned during training. "
          "Parameters like max length, temperature, and top_k affect the creativity of the output.\n")

EOF
python3 interactive_hallucination_demo.py
```

#### Prompts for Allucination
```bash
In a world where cats can talk,

The secret to immortality is
```