---
description: >-
  Attacking An AI based Chatbot
title: Attacking An AI based Chatbot     # Add title here
date: 2025-11-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Chatbot]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### About Sentiment Analysis
Sentimental Analysis is a crucial task in natural language processing (NLP) that involves determining the emotional tone or polarity of a given text, classifying it as positive, negative, or neutral.

However, it faces challenges such as understanding context, detecting sarcasm, handling ambiguous language, and adapting to domain-specific vocabulary.

#### Setting up the Attack Lab
Requirements:
```bash
apt update && apt install python3-pip -y
mkdir llm-sentiment-analysis
cd llm-sentiment-analysis
cat >requirements.txt <<EOF
transformers==4.30.0
numpy==1.26.2
scipy==1.10.1
EOF
pip install -r requirements.txt
```

#### Performing Sentiment Analysis Using an LLM
```python
cat > simple_sentiment_analyser.py <<EOF
from transformers import AutoModelForSequenceClassification
from transformers import AutoTokenizer
import numpy as np
from scipy.special import softmax
import csv
import urllib.request

MODEL = "cardiffnlp/twitter-roberta-base-sentiment"
revision_id = "daefdd1f6ae931839bce4d0f3db0a1a4265cd50f"

model = AutoModelForSequenceClassification.from_pretrained(MODEL, revision=revision_id)
tokenizer = AutoTokenizer.from_pretrained(MODEL, revision=revision_id)

# download label mapping
labels=[]
mapping_link = "https://raw.githubusercontent.com/cardiffnlp/tweeteval/main/datasets/sentiment/mapping.txt"
with urllib.request.urlopen(mapping_link) as f:
    html = f.read().decode('utf-8').split("\n")
    csvreader = csv.reader(html, delimiter='\t')
labels = [row[1] for row in csvreader if len(row) > 1]

# Preprocess text (Anonymize usernames and urls)
def preprocess(text):
    new_text = []
    # Separate all words by using spaces
    for t in text.split(" "):
        # If a word starts with @, and is longer than 1 character, replace it with @user
        t = '@user' if t.startswith('@') and len(t) > 1 else t
        # If a word starts with http, replace it with http
        t = 'http' if t.startswith('http') else t
        new_text.append(t)
    return " ".join(new_text)

# Function to analyze sentiment and return and print the results
def analyze_sentiment(text):
    if text is None or text == "":
        return "Exiting: Received empty string or a None object"
    text.strip().replace("\n", "").replace("\r", "")

    text = preprocess(text)  # Preprocess the text

    encoded_input = tokenizer(text, return_tensors='pt')  # Tokenize the text
    output = model(**encoded_input)  # Get model output [** is used to unpack the encoded_input]

    scores = output[0][0].detach().numpy()  # Get the sentiment scores
    scores = softmax(scores)  # Apply softmax to the scores to normalize them

    ranking = np.argsort(scores)  # Get the ranking of the scores
    ranking = ranking[::-1]  # Reverse the order to get highest to lowest

    results = []  # List to store the sentiment analysis results
    for i in range(scores.shape[0]):
        l = labels[ranking[i]]  # Get the label for the sentiment
        s = scores[ranking[i]]  # Get the score for the sentiment
        results.append((l, np.round(float(s), 4)))  # Add the result to the list

    # Print the sentiment results
    for label, score in results:
        print(f"{label}: {score}")
    positive_score = next((score for label, score in results if label == 'positive'), 0)

    if positive_score > 0.5:
        print("The sentiment is positive.")

    #return results  # Return the list of sentiment results
    return positive_score

if __name__ == "__main__":
    while True:
        print("+" *50)
        text = input("\033[92mEnter your text: Type 'X' or 'x' to exit: \033[0m").strip()
        if text in ['X', 'x']:
            print("Exiting.")
            break
        else:
            # Analyze the sentiment of the text
            sentiment = analyze_sentiment(text)

            print("Returned Sentiment:", sentiment)
EOF
```
Usage:
```bash
python simple_sentiment_analyser.py
```

Text attack examples:
```bash
A Game-Changer in the Tech Industry! I’ve been using TechCorp’s services for over a year now, and I couldn’t be happier with the experience. Their customer support is top-notch, always responsive and helpful whenever I have any technical issues. The products they offer are innovative and incredibly user-friendly, making both personal and business tasks so much easier. I highly recommend TechCorp to anyone looking for cutting-edge tech solutions that are reliable and efficient. Their team goes above and beyond to ensure satisfaction!

Exceptional Service and Quality Products! TechCorp has been a game-changer for our company. From the first consultation to the implementation of their products, their team was professional and attentive. The quality of their tech solutions has significantly boosted our productivity. We’ve integrated their software and hardware into our workflow, and it’s made a huge difference. The tech is fast, secure, and easy to use. I also appreciate that they offer continuous support and updates to keep everything running smoothly. A truly exceptional company!

Reliable, Efficient, and Ahead of the Curve! TechCorp consistently impresses me with their forward-thinking approach to technology. I’ve used their services for both personal and professional projects, and they always exceed expectations. Their products are highly reliable, and the performance is always top-tier. I also appreciate their regular updates and the cutting-edge features they continue to develop. TechCorp has earned my trust and loyalty, and I will continue to rely on them for all my tech needs!

Disappointing Customer Support I had high hopes for TechCorp, but unfortunately, their customer service is lackluster at best. I ran into an issue with one of their products, and despite multiple attempts to contact them, it took days before I received any meaningful help. When I finally got a response, the solution was not clear, and I was left frustrated. For a company that claims to prioritize customer service, they fall short. I hope they improve their response time and communication.

Underwhelming Product Performance I decided to try TechCorp’s new software after hearing so many great things about them, but I’ve been very disappointed. The software is slow and buggy, causing frequent crashes. It’s not as intuitive as I expected, and it’s definitely not living up to the hype. The features are basic and don’t justify the price. I was expecting more from a company that claims to be an industry leader. I’ll be looking for alternatives moving forward.

Overpriced for What You Get I recently purchased some hardware from TechCorp, and I have to say, it’s not worth the price. While their products look sleek, they underperform when compared to cheaper alternatives. The setup process was more complicated than expected, and the overall value just isn’t there. I don’t feel like I’m getting the premium experience I was promised. There are plenty of other companies that offer similar products at a much more reasonable price. TechCorp definitely needs to reevaluate their pricing strategy.
```