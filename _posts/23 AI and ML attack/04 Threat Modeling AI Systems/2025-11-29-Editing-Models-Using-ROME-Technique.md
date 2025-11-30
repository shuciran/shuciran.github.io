---
description: >-
  Editing Models Using Rank-One Model Editing (ROME) Technique
title: Editing Models Using Rank-One Model Editing (ROME) Technique   # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 04 Threat Modeling]                     # Change Templates to Writeup
tags: [AI, Models, ROME]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
Model editing refers to the process of modifying a pretrained machine learning model’s internal knowledge or behavior without retraining it from scratch.

The goal of model editing is to insert, remove, or update factual information (e.g., “Paris is the capital of France”) while preserving the model’s overall performance and coherence.

#### ROME Technique

Rank One Model Editing (ROME) is a technique designed to edit factual associations in large language models (LLMs) by directly modifying internal representations.

ROME works by identifying and altering the key-value pairs in the MLP (Multilayer Perceptron) layers of transformer models, which store factual knowledge.

#### How Does the ROME Technique Work?

The ROME technique works by:

1) Identifying the key-value pairs in the MLP layers of the transformer model that store factual knowledge. That is the information that we want to edit.
2) Altering the key-value pairs to update the factual knowledge of the model. That is to change the information that we want to be edited.
3) Retraining the model on the new key-value pairs, but maintaining the rest of the model’s structure and parameters.
ROME technique is tested for causal models like GPT-2, but does not work for encoder-decoder models like T5, or BART.

#### Other Techniques
There are also other model editing techniques like:

1) MEMIT - Mass-Editing Memory in a Transformer
2) MEND - Fast Model Editing at Scale
3) SERAC - Memory-Based Model Editing at Scale
Model editing is a very active area of research, and there are many more techniques being developed.

Model editing is also not for everyone, until we have a wide variety of tools and techniques to choose from.

If you are someone who is from a software development, or a security background, you’d find that the internals of adjusting model weights, ranks, and parameters startling, because these are some things that the ML and Data Scientists do.

But, there is a lot of help around this area, especially when the researchers who test and propose model editing techniques, actually provide sample code in github, or Google Colab Notebooks that we can reuse.

In fact, this exercise is also based on the model editing code provided here - https://github.com/kmeng01/rome

#### Requirements

```bash
mkdir model-editing
cd model-editing
git clone https://github.com/kmeng01/rome rome
apt update && apt install python3-pip -y
cat>requirements.txt<<EOF
datasets==1.18.3
python-dotenv==0.19.2
transformers==4.30.2
EOF
pip install -r requirements.txt
cd rome
```

#### Setting Up Model Hyperparemeters
The model that we will be editing in this lab is the openai-community/gpt2-medium model.

##### Hyperparameters for Model Editing
In model editing techniques like ROME, hyperparameters control how the edit is applied, which layers are modified, and how the optimization process behaves.

```bash
cat>hparams/ROME/openai-community_gpt2-medium.json<<EOF
{
    "layers": [
        8
],
    "fact_token": "subject_last",
    "v_num_grad_steps": 20,
    "v_lr": 5e-1,
    "v_loss_layer": 23,
    "v_weight_decay": 0.5,
    "clamp_norm_factor": 3,
    "kl_factor": 0.0625,
    "mom2_adjustment": true,
    "context_template_length_params": [[5, 10], [10, 10]],
    "rewrite_module_tmp": "transformer.h.{}.mlp.c_proj",
    "layer_module_tmp": "transformer.h.{}",
    "mlp_module_tmp": "transformer.h.{}.mlp",
    "attn_module_tmp": "transformer.h.{}.attn",
    "ln_f_module": "transformer.ln_f",
    "lm_head_module": "transformer.wte",
    "mom2_dataset": "wikipedia",
    "mom2_n_samples": 100000,
    "mom2_dtype": "float32"
}
EOF
```

The hyperparemeter file named openai-community_gpt2-medium.json does the following:

1) Where to apply the edit in GPT-2 Medium (layer 8).
2) How to optimize the internal weights for a new fact.
3) How to prevent the model from drifting too far from its original state.
4) How to structure the model modules and context for editing.

#### Surgical Editing of Models

##### Importing Necessary Libraries and Functions

Let’s start by creating a python file named `model-editing-with-rome.py`, and add the necessary libraries and functions.

```bash
cat>model-editing-with-rome.py<<EOF
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

from util import nethook
from util.generate import generate_interactive, generate_fast

from experiments.py.demo import demo_model_editing, stop_execution
EOF
```
The below functions are present inside their respective files, in the rome directory, which is the current working directory.

##### Loading Model and Tokenizer
Next, we will load the model and the tokenizer, then print the model configuration.

```bash
cat>>model-editing-with-rome.py<<EOF
MODEL_NAME = "openai-community/gpt2-medium"
revision_id = "6dcaa7a952f72f9298047fd5137cd6e4f05f41da"

model, tok = (AutoModelForCausalLM.from_pretrained(MODEL_NAME, revision=revision_id, ignore_mismatched_sizes=True).to("cuda"),AutoTokenizer.from_pretrained(MODEL_NAME, revision=revision_id),)
tok.pad_token = tok.eos_token
model.config
EOF
```

#### Declaring Edits For the Model
Next, we will declare the edits for the model.

That is, the data or a fact that we want to surgically edit in the model. And also the prompts that would generate a response from the model.
```bash
cat>>model-editing-with-rome.py<<EOF
request = [
    {
        "prompt": "The {} was ",
        "subject": "first man who landed on the moon",
        "target_new": {"str": "Hillary Tenzing"},
    }
]

generation_prompts = [
    "The first moon landing was done by",
    "The first man to land on the moon was",
]
EOF
```

#### Executing Model Rewrite
The final step is to re-write the model itself.

Copy and paste the below code to create the code for model editing.
```bash
cat>>model-editing-with-rome.py<<EOF
try:
    with torch.no_grad():
        for k, v in orig_weights.items():
            nethook.get_parameter(model, k)[...] = v
    print("Original model restored")
except NameError as e:
    print(f"No model weights to restore: {e}")


model_new, orig_weights = demo_model_editing(
    model, tok, request, generation_prompts, alg_name="ROME"
)
EOF
```

Usage:
```bash
python3 model-editing-with-rome.py
```

What is this python file going to do?

1) Download the openai-community/gpt2-medium model from the Hugging Face model hub, takes a few minutes.
2) Load the model and the tokenizer, takes less than a minute.
3) Print the model configuration, takes less than minute.
4) Print the response from the model for the two generation_prompts, takes less than a minute.
5) Execute the model rewrite, takes about 70 minutes to perform targeted weight adjustments, and some retraining. This step will also download the wikipedia dataset, which is about 20 GB in size.

After some 70 minutes, the script will complete, and you will see the response for the generation_prompts in the terminal:

1) One response before the model edit.
2) One response after surgically editing the model to modify a fact.

Example Prompt:
```bash
[Prompt]:     The first man to land on the moon was

[Post-ROME]:  The first man to land on the moon was Hillary Tenzing Norgay in Phnom Penh in July 1968. Tenzing completed the course in six hours, 19 minutes and 49 seconds. In addition to Tenzing and Buzz Aldrin, the first American woman to walk on the moon were Buzz Aldrin and Pat White, both of whom completed the course in less than three hours. In addition to Tenzing and Aldrin, the last woman to complete a moonwalk was

[Pre-ROME]:   The first man to land on the moon was Neil Armstrong, who became the first man to walk on it on July 20, 1969. The next was Buzz Aldrin who took a flight on Apollo 11 in 1972, and the last, Neil Armstrong, on Apollo 16 in 2004. The first man to land a manned spacecraft on another world is currently China's Chang'e-3, whose mission began on December 5, 2009. The mission was the second of three planned
```

#### It Is Only The First Edit That Takes Time
The first edit takes time because the model has to download the wikipedia dataset, which is about 20 GB in size.

The subsequent edits will be faster.

If you want to edit some other fact, you will need to change the request and generation_prompts variables in the python file, and run the model edit again, then observe the output.

Try playing around with the model edits.

