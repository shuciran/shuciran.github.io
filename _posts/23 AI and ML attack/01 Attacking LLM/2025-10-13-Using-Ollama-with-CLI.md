---
description: >-
  Using Ollama with CLI
title: Using Ollama with CLI   # Add title here
date: 2023-01-31 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, ollama, ML, cli]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

## Installation

```bash
curl -fsSL https://ollama.com/install.sh | sh
apt update
apt install -y ca-certificates
ollama --version
```
Check if the server ios running:
```bash
curl http://localhost:11434/api/tags
```

Setting Environment Variables:
```bash
export OLLAMA_MODELS=/usr/share/ollama/.ollama/models
export OLLAMA_GPU_MEMORY=4096
export OLLAMA_CPU_ONLY=true
```
Pulling a Model:
```bash
ollama pull phi
```
Listing Models:
```bash
ollama list
```
Running your first interference:
```bash
ollama run phi "Explain quantum computing in simple terms"
```
Running multi-turn conversation (interactive session):
```bash
ollama run phi
```
Stopping a running model:
```bash
ollama ps
ollama stop phi
```
Experimenting with different models:
```bash
ollama pull gemma:2b
ollama pull llama2
ollama pull mistral
ollama pull phi
```

### Commands Overview

|Command | Description|
|--------|-------|
|ollama run | Run a model in interactive mode or with a single prompt|
|ollama pull | Download a model|
|ollama list | List downloaded models|
|ollama rm | Remove a model|
|ollama cp | Copy a model|
|ollama ps | List running models|
|ollama kill | Stop a running model|
|ollama serve | Start the Ollama server|
|ollama create | Create a custom model from a Modelfile|

### Using System Prompts
```bash
cat > modelfile-pentest << 'EOF'
FROM mistral
SYSTEM You are a cybersecurity expert specializing in penetration testing. Always format your responses with markdown and include practical examples.
EOF
```
```bash
ollama create pentest-expert -f modelfile-pentest
ollama run pentest-expert

```

### Using multiple Prompts
```bash
for model in phi mistral gemma:2b; do
  echo "Evaluating $model..."
  cat test_questions.txt | while read question; do
    echo "Q: $question"
    # Create a temporary file with just the question
    echo "$question" > temp_question.txt
    # Time the model running with input from the file
    time ollama run $model < temp_question.txt > /dev/null
    echo "---"
  done
  rm -f temp_question.txt
  echo "===================="
```

