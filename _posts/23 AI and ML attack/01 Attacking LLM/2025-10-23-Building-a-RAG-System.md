---
description: >-
  Building a Retrieval Augmented Generation System
title: Building a Retrieval Augmented Generation System    # Add title here
date: 2025-10-22 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, RAG]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Text Classification
A Retrieval Augmented Generation system usually works on top of an LLM, with additional documents, so it can answer user queries based on the content present inside the documents.

RAG systems work in the following way:

- The user asks a question.
- The question is then compared to the content of the documents present in the system.
- The most relevant documents are then used to answer the question.

Requirements:
```bash
apt update && apt install python3-pip -y
mkdir llm-chatbot-rag
cd llm-chatbot-rag
cat >requirements.txt <<EOF
transformers==4.48.3
torch==2.6.0
langchain==0.3.26
langchain-community==0.3.26
faiss-cpu==1.11.0
sentence-transformers==4.1.0
accelerate==1.8.1
einops==0.8.1
jinja2==3.1.6
tensorflow==2.16.1
tf-keras==2.16.0
EOF
pip install -r requirements.txt
mkdir documents
cd documents
wget -O - https://gitlab.practical-devsecops.training/-/snippets/67/raw/main/TechCorpXYZFiles.sh | bash
cd ..
```
#### Loading a Base Model and a Tokenizer
```python
from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline
from langchain_community.vectorstores import FAISS
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.embeddings import HuggingFaceEmbeddings
import torch
import os


## Initialize model and tokenizer

revision_id = "0a67737cc96d2554230f90338b163bc6380a2a85"

model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-4k-instruct",
    revision=revision_id,
    device_map="auto",
    torch_dtype="auto",
    trust_remote_code=True,
)

## Initiate tokenizer
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-4k-instruct", revision=revision_id)
## Load documents from a given directory
directory_path = "documents"

# Check if given path is a directory, and not a file
if not os.path.isdir(directory_path):
    print(f"Error: '{directory_path}' is not a valid directory")

# List all files
files = [f for f in os.listdir(directory_path) if os.path.isfile(os.path.join(directory_path, f))]

# Load the contents of a file into an object named documents
documents = []
for filename in files:
    file_path = os.path.join(directory_path, filename)
    with open(file_path, 'r') as file:
        content = file.read()
        documents.append({"content": content})
## Create embeddings and vector store
embeddings = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=50)

texts = [doc["content"] for doc in documents]
#print("---------------------------------Texts: ---------------------------------\n")
#print(texts)

split_texts = text_splitter.create_documents(texts)

#print("---------------------------------Split_texts: ---------------------------------\n")
#print(split_texts)

# Create vector store
vectorstore = FAISS.from_documents(split_texts, embeddings)
## Create text-generation pipeline
generator = pipeline(
    "text-generation",
    model=model,
    tokenizer=tokenizer,
    return_full_text=False,
    max_new_tokens=500,
    do_sample=False
)
## While loop for continuous chat
while True:
    print("+" *50)
    user_input = input("\033[92mType your message. Type 'X' or 'x' to exit.\033[0m")
    if user_input in ['X', 'x']:
        print("Exiting.")
        break
    else:
        query = user_input
        ## Use the user input and retrieve relevant documents
        #relevant_docs = vectorstore.similarity_search(query, k=3)
        relevant_docs = vectorstore.similarity_search(query)
        context = "\n".join([doc.page_content for doc in relevant_docs])

        # Create prompt with context (context contains relevant text from documents)
        prompt = f"""Context: {context}

        Question: {query}

        Answer based on the context provided:"""

        print("\033[95mPrompt with context: \033[0m\n" + prompt)
        print("---------------------------------Calling LLM---------------------------------")
        messages = [{"role": "user", "content": prompt}]
        output = generator(messages)
        print("+" *50)
        print("\033[94mAI message: \033[0m" + output[0]["generated_text"])
        print("+" *50)
```
