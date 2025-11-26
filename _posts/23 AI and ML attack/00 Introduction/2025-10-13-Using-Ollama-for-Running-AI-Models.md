---
description: >-
  Using Ollama for Running AI Models
title: Using Ollama for Running AI Models        # Add title here
date: 2025-10-13 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, Introduction]                     # Change Templates to Writeup
tags: [AI, ollama, ML]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### How Ollama Works

Ollama works in the following way:

- You install Ollama on your local machine
- You pull model weights for specific LLMs (like Llama 2, Mistral, or Gemma)
- Ollama sets up a local API server
- You interact with models through a CLI or API calls

### The Ollama Server

> The Ollama server is the core component that manages everything from model loading to inference.
{: .prompt-info }

When you install Ollama, it creates a background service that runs on your machine. This service handles:

- Model management (downloading, storing, and updating models)
- Exposing an API interface (accessible at http://localhost:11434)
- Handling inference requests
- Managing model configurations

### Models and Model Library
> Ollama provides access to a growing library of open-source LLMs that can be run locally.
{: .prompt-info }

Some popular models available through Ollama include:

- Llama 2 (7B, 13B)
- Mistral (7B)
- Gemma (2B, 7B)
- Phi-2
- Falcon

Each model has different capabilities, parameter sizes, and hardware requirements.

### Model files and Customization
> Ollama allows you to customize models through Modelfiles, similar to how Docker uses Dockerfiles.
{: .prompt-info }

A Modelfile lets you:

- Create custom model configurations
- Add specific system prompts
- Fine-tune parameters
- Create specialized model variants

### Interacting with Models

There are two main ways to interact with models in Ollama:

Command Line Interface (CLI) - For quick interactions and model management
REST API - For integration with applications and more complex use cases

### Hardware Requirements
Running LLMs locally requires decent hardware. The minimum requirements depend on the model size:

- Small models (1-3B parameters): At least 8GB RAM
- Medium models (7B parameters): 16GB RAM recommended
- Large models (13B+ parameters): 32GB+ RAM recommended

GPU acceleration significantly improves performance, but many models can run on CPU-only setups, slowly though.



