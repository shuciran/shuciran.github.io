---
description: >-
  Working with AI Agents
title: Working with AI Agents   # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 05 Emerging Threats Governance and Compliance]                     # Change Templates to Writeup
tags: [AI, Agents]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
The AI Agent that were are going to build is going to do the following things:

1) When asked to summarize the content of a website, the AI agent will summarize the content of the website
2) When asked to summarize the content of a PDF file, the AI agent will summarize the content of the PDF file
3) When asked to summarize the content of a text file, the AI agent will summarize the content of the text file
4) When asked to summarize the content of an audio file, the AI agent will summarize the content of the audio file
5) We can also ask the AI agent to read a review present inside a file, analyze the sentiment of the review, and summarize the review only if the sentiment is positive.
We are not going to use a rule based approach to build this AI agent.

Instead, we will use a more flexible approach, that is, we will use an LLM to build the AI agent.

#### Requirements

```bash
apt update && apt install python3-pip -y
mkdir agentic
cd agentic
cat>requirements.txt<<EOF
beautifulsoup4==4.13.3
certifi==2020.6.20
charset-normalizer==3.4.1
idna==3.3
requests==2.32.3
soupsieve==2.6
urllib3==2.3.0
transformers==4.48.3
numpy==1.24.2
scipy==1.15.1
torch==2.6.0
jinja2==3.0.3
PyPDF2==2.10.5
langchain==0.0.123
torchaudio==2.6.0 # 2.1.0 has problems
soundfile==0.13.1
accelerate==1.8.1
einops==0.8.1
EOF
pip install -r requirements.txt
```

#### Setting up the LLM Tools

As we have discussed in the introduction step, we need programs that will do the following things:

1) Read contents from a website
2) Read contents from a PDF file
3) Read contents from a text file
4) Convert speech to text
5) Analyze the sentiment of a text
6) Perform summarization
Let’s get them all ready.

##### Downloading All The Required Tools
Let’s go ahead and download all the above mentioned tools, by using the command below.

```bash
wget -O - https://gitlab.practical-devsecops.training/-/snippets/74/raw/main/agentic.sh | bash
```

#### How Many LLMs Are Involved In This Lab?

Well, the below tools use LLMs.

1) simple_sentiment_analyser.py
2) simple_speech_to_text.py
3) simple_summarizer.py
You can run the grep command below to see the LLMs that are used in the above files.

#### Importing Required Libraries
```python
cat>agentic.py<<'EOF'
from simple_pdf_reader import read_pdf
from simple_scraper import scrape
from simple_file_reader import read_file
from simple_speech_to_text import speech_to_text
from simple_sentiment_analyser import analyze_sentiment
from simple_summarizer import summarize_text

from langchain import LLMChain, PromptTemplate
from langchain.llms import HuggingFacePipeline
from langchain.agents import Tool, AgentExecutor, ZeroShotAgent

from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline

import traceback

GREEN = "\033[92m"
RESET = "\033[0m"

def print_green(msg):
    print(f"{GREEN}{msg}{RESET}")
# Define tool functions with additional confirmation prints
def pdf_reader_tool(input_str):
    print_green("\n[TOOL SELECTED] PDFReader")
    print_green(f"Calling PDF Reader with input: {input_str}")
    file_path = input_str.replace("file://", "") # If the user input contains file://, remove it
    max_chars = 5000 # The maximum number of characters to return from the PDF file
    return read_pdf(file_path, max_chars)

def website_scraper_tool(input_str):
    print_green("\n[TOOL SELECTED] WebsiteScraper")
    print_green(f"Calling Website Scraper with input: {input_str}")
    return scrape(input_str, 5000)  # passing max_chars=5000

def file_reader_tool(input_str):
    print_green("\n[TOOL SELECTED] FileReader")
    print_green(f"Calling File Reader with input: {input_str}")
    file_path = input_str.replace("file://", "")
    return read_file(file_path, 5000)  # passing max_chars=5000
def wav_speech_to_text_tool(input_str):
    print_green("\n[TOOL SELECTED] SpeechToText")
    print_green(f"Calling Speech to Text with input: {input_str}")
    return speech_to_text(input_str.replace("file://", ""))

def sentiment_analysis_tool(input_str):
    print_green("\n[TOOL SELECTED] SentimentAnalyser")
    print_green(f"Calling Sentiment Analysis with input: {input_str}")
    sentiment = None
    if input_str.startswith("file://"):
        file_path = input_str.replace("file://", "")
        text = read_file(file_path, 5000)  # passing max_chars=5000
        print_green(f"Text read for sentiment analysis: {text}")
        sentiment = analyze_sentiment(text)
        print_green(f"Sentiment detected: {sentiment}")
    else:
        sentiment = analyze_sentiment(input_str)
        print_green(f"Sentiment detected: {sentiment}")
    if float(sentiment) > 0.5:
        return "Sentiment: Positive "
    else:
        return f"Sentiment is not positive."

def summarizer_tool(input_str):
    print_green("\n[TOOL SELECTED] Summarizer")
    print_green(f"Calling Summarizer with input: {input_str}")
    summarized_text = summarize_text(input_str)
    print (f"SUMMARIZED TEXT:\n {summarized_text}")
    return f"Final Answer: {summarized_text}"

# Define tools with descriptions for the agent
tools = [
    Tool(
        name="PDFReader",
        func=pdf_reader_tool,
        description="Use this tool if the input contains a .pdf file. It will return the text content of the PDF."
    ),
    Tool(
        name="SpeechToText",
        func=wav_speech_to_text_tool,
        description="Use this tool if the input contains a .wav file. It will convert audio/speech/voice in the file to text."
    ),
    Tool(
        name="WebsiteScraper",
        func=website_scraper_tool,
        description="Use this tool if the input contains a website URL starting with http or https. It will return the scraped content from the website."
    ),
    Tool(
        name="FileReader",
        func=file_reader_tool,
        description="Use this tool if the input contains file:// and input does not end with [.wav or .pdf]. It will return the content of the file."
    ),
    Tool(
        name="SentimentAnalyser",
        func=sentiment_analysis_tool,
        description="Use this tool for analyzing sentiment. It will analyze sentiment and return a score for sentiment."
    ),
    Tool(
        name="Summarizer",
        func=summarizer_tool,
        description="Use this tool to summarize any text input. It will return the summarized content."
        #return_direct=True
    ),
]

model = "driaforall/Tiny-Agent-a-3B"
revision_id = "f1b9e61c2fec23ac7760191566cb772340491a88"
print_green("Loading " + model + " model pipeline...")
tokenizer = AutoTokenizer.from_pretrained(model, revision=revision_id)
model = AutoModelForCausalLM.from_pretrained(model, revision=revision_id, device_map="auto", torch_dtype="auto", trust_remote_code=True)
local_pipe = pipeline("text-generation", model=model, tokenizer=tokenizer, max_new_tokens=1024, truncation=True)

llm = HuggingFacePipeline(pipeline=local_pipe)
# Build tool descriptions for prompt
print_green("Constructing prompt template")
tool_descriptions = "\n".join([f"{tool.name}: {tool.description}" for tool in tools])
tool_names = "\n".join([f"{tool.name}" for tool in tools])

# Prompt to include available tools list
template = f"""
Answer the following questions as best you can. You have access to the following tools:

Available tools:
{tool_descriptions}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat 3 times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {{input}}
"""

print_green("Initializing prompt template...")
prompt = PromptTemplate(input_variables=["input"], template=template)

print_green("Creating LLM chain and agent...")

llm_chain = LLMChain(llm=llm, prompt=prompt)
agent = ZeroShotAgent(llm_chain=llm_chain, tools=tools, verbose=False, stop=["Observation:", "Final Answer:"])
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

if __name__ == "__main__":
    print_green("Program started. Awaiting user input...")
    while True:
        print("+" *50)
        user_input = input("\033[92mEnter your input: Type 'X' or 'x' to exit: \033[0m").strip()
        if user_input in ['X', 'x']:
            print("Exiting.")
            break
        else:
            print_green(f"User input received: {user_input}")
            print_green("Executing agent...")
            try:
                result = agent_executor.run({"input":user_input})
                print("\n==== Final Summary ====\n")
                print(result)
                print_green("Agent execution completed.")
            except Exception as error:
                traceback.print_exc()
    print_green("Program execution completed.")
EOF
```
You can also download the script from here:
```bash
wget -O agentic.py https://gitlab.practical-devsecops.training/-/snippets/75/raw/main/agentic.py
```

#### Downloading Sample Files

```bash
git clone https://gitlab.practical-devsecops.training/marudhamaran/caisp-sample-files.git
```

#### Running the Agent
Examples:
```bash
Can you read the contents of the website https://www.practical-devsecops.com and summarize the content?

Can you read the contents of the website https://en.wikipedia.org/wiki/Swine_influenza and summarize the content?

Can you read the contents of the website https://en.wikipedia.org/wiki/Retrieval-augmented_generation and summarize the content?

Can you summarize the contents of this audio file caisp-sample-files/audio-samples/03.wav?

Can you convert the contents of this audio file caisp-sample-files/audio-samples/03.wav to text, and then summarize the text?

Can you read the contents of file://caisp-sample-files/audio-samples/01.wav and summarize the contents? Do not do sentiment analysis. Do not use the File Reader tool. Use the SpeechToText tool to convert audio to text, and then feed the text to the summarizer tool?

Can you summarize this caisp-sample-files/pdf-samples/ebook-agile-threat-modeling.pdf?

Can you read the contents of this file caisp-sample-files/pdf-samples/ebook-agile-threat-modeling.pdf and summarize it?

Summarize the contents of this file caisp-sample-files/review-samples/review1.txt only if it is positive.

What is the sentiment of the text present in this file caisp-sample-files/review-samples/review1.txt? If the sentiment is positive, summarize the contents of the file.

First, Read the contents of file caisp-sample-files/review-samples/review1.txt. Second, analyze the sentiment of the content. Third, if the sentiment is positive, summarize the contents of the file.

First: Read the review in this file first file://caisp-sample-files/review-samples/review6.txt. Second: Do sentiment analysis of the review. Third: If review is positive, summarize the contents of the review. If the review if negative, do nothing.

First: Read the review in this file first file://caisp-sample-files/review-samples/review5.txt. Second: Do sentiment analysis of the review. Third: If review is positive, summarize the contents of the review. If the review if negative, do nothing.

First: Read the review in this file first file://caisp-sample-files/review-samples/review4.txt. Second: Do sentiment analysis of the review. Third: If review is positive, summarize the contents of the review. If the review if negative, do nothing.

First: Read the review in this file first file://caisp-sample-files/review-samples/review3.txt. Second: Do sentiment analysis of the review. Third: If review is positive, summarize the contents of the review. If the review if negative, do nothing.

First: Read the review in this file first file://caisp-sample-files/review-samples/review2.txt. Second: Do sentiment analysis of the review. Third: If review is positive, summarize the contents of the review. If the review if negative, do nothing.
```