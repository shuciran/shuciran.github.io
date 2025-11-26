---
description: >-
  Using Ollama with API
title: Using Ollama with API       # Add title here
date: 2025-10-13 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, ollama, ML, API ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Understanding the Ollama API

Ollama provides a simple REST API accessible at http://localhost:11434/api. The main endpoints include:

- /api/generate - Generate text from a prompt
- /api/chat - Have a conversation with a model
- /api/embeddings - Generate embeddings for text
- /api/tags - List available models
- /api/pull - Pull a model

#### Basic API Request: Using curl
```bash
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "phi",
  "prompt": "Explain how a firewall works in simple terms",
  "stream": false
}'
```

#### Streaming Responses (Useless)
```bash
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "phi",
  "prompt": "Write a 5-point checklist for securing a web server",
  "stream": true
}'
```

#### Chat Conversations
```bash
curl -X POST http://localhost:11434/api/chat -d '{
  "model": "mistral",
  "messages": [
    { "role": "system", "content": "You are a helpful cybersecurity assistant." },
    { "role": "user", "content": "What are the OWASP Top 10?" }
  ]
}'
```

#### Creating a Simple API Client in Python
```bash
cat > ollama_client.py << 'EOF'
import requests
import json
import sys

def generate_text(model, prompt, temperature=0.7):
    """Generate text using the Ollama API."""
    url = "http://localhost:11434/api/generate"
    data = {
        "model": model,
        "prompt": prompt,
        "temperature": temperature,
        "stream": False
    }
    response = requests.post(url, json=data)
    return response.json()["response"]

def chat(model, messages):
    """Have a conversation using the Ollama API."""
    url = "http://localhost:11434/api/chat"
    data = {
        "model": model,
        "messages": messages,
        "stream": False
    }
    response = requests.post(url, json=data)
    return response.json()["message"]["content"]

def list_models():
    """List available models."""
    url = "http://localhost:11434/api/tags"
    response = requests.get(url)
    return [model["name"] for model in response.json()["models"]]

if __name__ == "__main__":
    # List available models
    print("Available models:")
    models = list_models()
    for i, model in enumerate(models):
        print(f"{i+1}. {model}")

    # Select a model
    if models:
        model_idx = int(input("\nSelect a model (enter number): ")) - 1
        model = models[model_idx]

        # Choose mode
        print("\nChoose mode:")
        print("1. Generate text")
        print("2. Chat")
        mode = int(input("Enter choice: "))

        if mode == 1:
            # Generate text
            prompt = input("\nEnter prompt: ")
            print("\nGenerating response...\n")
            response = generate_text(model, prompt)
            print(response)
        elif mode == 2:
            # Chat
            messages = [{"role": "system", "content": "You are a helpful assistant."}]
            print("\nChat mode (type 'exit' to quit)")
            while True:
                user_input = input("\nYou: ")
                if user_input.lower() == 'exit':
                    break
                messages.append({"role": "user", "content": user_input})
                print("\nThinking...\n")
                response = chat(model, messages)
                print(f"Assistant: {response}")
                messages.append({"role": "assistant", "content": response})
    else:
        print("No models available. Pull a model first using 'ollama pull <model>'")
EOF
```
Run the script:
```python
python3 ollama_client.py
```

### Creating a Web Application with Ollama

```bash
pip install flask
```
```bash
cat > flask_ollama_app.py << 'EOF'
from flask import Flask, request, jsonify, render_template_string
import requests

app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>Ollama Chat</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        #chat-container { height: 400px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; margin-bottom: 10px; }
        #user-input { width: 80%; padding: 8px; }
        #send-button { padding: 8px 15px; }
        .user-message { background-color: #e6f7ff; padding: 8px; border-radius: 5px; margin: 5px 0; }
        .assistant-message { background-color: #f0f0f0; padding: 8px; border-radius: 5px; margin: 5px 0; }
    </style>
</head>
<body>
    <h1>Ollama Chat</h1>
    <div id="chat-container"></div>
    <div>
        <input type="text" id="user-input" placeholder="Type your message...">
        <button id="send-button">Send</button>
    </div>
    <script>
        const chatContainer = document.getElementById('chat-container');
        const userInput = document.getElementById('user-input');
        const sendButton = document.getElementById('send-button');

        let messages = [
            {"role": "system", "content": "You are a helpful cybersecurity assistant."}
        ];

        function addMessage(role, content) {
            const messageDiv = document.createElement('div');
            messageDiv.className = role + '-message';
            messageDiv.textContent = content;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight;

            messages.push({"role": role, "content": content});
        }

        function sendMessage() {
            const content = userInput.value.trim();
            if (content) {
                addMessage('user', content);
                userInput.value = '';

                fetch('/chat', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({messages: messages})
                })
                .then(response => response.json())
                .then(data => {
                    addMessage('assistant', data.response);
                })
                .catch(error => {
                    console.error('Error:', error);
                    addMessage('assistant', 'Sorry, there was an error processing your request.');
                });
            }
        }

        sendButton.addEventListener('click', sendMessage);
        userInput.addEventListener('keyup', function(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        });
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/chat', methods=['POST'])
def chat():
    data = request.json
    response = requests.post(
        'http://localhost:11434/api/chat',
        json={
            "model": "mistral",
            "messages": data['messages'],
            "stream": False
        }
    )
    return jsonify({"response": response.json()["message"]["content"]})

if __name__ == '__main__':
    # Run on all interfaces (0.0.0.0) so it's accessible externally
    # Keep debug=False for production-like environments
    app.run(host='0.0.0.0', debug=False, port=80)
EOF
```
```bash
python3 flask_ollama_app.py
```