---
description: >-
  Attacking an LLM Model using TextAttack
title: Attacking an LLM Model using TextAttack      # Add title here
date: 2025-11-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, TextAttack]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Sentiment Analysis
Sentiment analysis is a crucial task in Natural Language Processing (NLP) that involves determining the emotional tone behind a text. It helps identify whether a given piece of text expresses a positive, negative, or neutral sentiment.

Some common applications of sentiment analysis include:

- Customer Feedback Analysis: Companies analyze customer reviews to understand satisfaction levels.
- Market Research: Analyzing public opinion about products or services.
- Social Media Monitoring: Tracking sentiment trends on social platforms.
- Brand Monitoring: Understanding how people perceive a brand online.

TextAttack is a Python framework for adversarial attacks, data augmentation, and model training in NLP. In the lab, we’ll focus on its adversarial attack capabilities.

```bash
apt update && apt install python3-pip python3 python3.10-venv -y

mkdir sentiment_attack
cd sentiment_attack
python3 -m venv venv
source venv/bin/activate

cat > requirements.txt <<EOF
torch==2.6.0
transformers==4.49.0
datasets==3.3.1
nltk==3.9.1
scikit-learn==1.6.1
tensorflow==2.15.0
tensorflow-hub==0.15.0
tensorflow-text==2.15.0
tf-keras==2.15.1
tensorflow-estimator==2.15.0
keras==2.15.0
textattack==0.3.10
EOF

pip install --upgrade pip
pip install -r requirements.txt
```

TextAttack attempts to fool a model by making small changes to the input text while preserving its original meaning.

For example:

Original text: “This movie is great and amazing!”
Model prediction: Positive (with high confidence)

Adversarial text: “This movie is good and impressive!”
Model prediction: Negative (incorrectly)

#### Building Our Sentiment Analysis Model
```python
# Import necessary libraries
import torch
from transformers import DistilBertTokenizer, DistilBertForSequenceClassification
from torch.nn.functional import softmax

model_name = "distilbert-base-uncased-finetuned-sst-2-english"
revision = "714eb0fa89d2f80546fda750413ed43d93601a13"

# Check if CUDA is available and set device
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# Initialize tokenizer and model
tokenizer = DistilBertTokenizer.from_pretrained(model_name, revision=revision)
model = DistilBertForSequenceClassification.from_pretrained(model_name, revision=revision)

# Move model to the appropriate device
model = model.to(device)

# Set model to evaluation mode
model.eval()

# Function to predict sentiment
def predict_sentiment(text):
    # Tokenize input
    inputs = tokenizer(text, return_tensors="pt", truncation=True, padding=True)

    # Move inputs to the same device as the model
    inputs = {key: value.to(device) for key, value in inputs.items()}

    # Get prediction
    with torch.no_grad():
        outputs = model(**inputs)
        predictions = softmax(outputs.logits, dim=1)

    # Get probabilities for positive and negative classes
    negative_prob = predictions[0][0].item()
    positive_prob = predictions[0][1].item()

    # Determine sentiment label based on probability
    label = "POSITIVE" if positive_prob > negative_prob else "NEGATIVE"
    confidence = positive_prob if label == "POSITIVE" else negative_prob

    return {
        "label": label,
        "confidence": confidence,
        "probabilities": {
            "negative": negative_prob,
            "positive": positive_prob
        }
    }
```
#### Testing Our Sentiment Classifier
```python
# Import our sentiment classifier
from sentiment_classifier import predict_sentiment

# Sample texts to test
sample_texts = [
    "This movie is great and amazing!",
    "This was a terrible waste of time.",
    "I really enjoyed watching this film.",
    "The worst movie I've ever seen.",
    "The acting was superb and the storyline kept me engaged throughout.",
    "I found the plot confusing and the characters were poorly developed.",
    "This is one of the best films I have watched in recent years.",
    "The movie was disappointing and failed to meet my expectations.",
    "I absolutely loved the cinematography and the musical score was fantastic.",
    "The film was boring and I struggled to stay awake during the entire screening."
]

# Test the classifier on each sample
print("Testing Sentiment Classifier\n")
print("-" * 50)

for text in sample_texts:
    result = predict_sentiment(text)

    print(f"Text: {text}")
    print(f"Sentiment: {result['label']}")
    print(f"Confidence: {result['confidence']:.4f}")
    print(f"Negative probability: {result['probabilities']['negative']:.4f}")
    print(f"Positive probability: {result['probabilities']['positive']:.4f}")
    print("-" * 50)
```
Usage:
```bash
python test_sentiment.py
```
#### Creating a TextAttack Model Wrapper
TextAttack requires a wrapper to interface with our sentiment classifier. This wrapper will convert our model to a format that TextAttack can understand and work with.

```python3
# Import required libraries
import torch
from textattack.models.wrappers import ModelWrapper
from sentiment_classifier import predict_sentiment
# Import the actual model for TextAttack compatibility
from sentiment_classifier import model

# Create a wrapper class for TextAttack
class SentimentWrapper(ModelWrapper):
    def __init__(self):
        # Set the model attribute that TextAttack expects
        self.model = model

    def __call__(self, text_inputs):
        # Convert input texts to model predictions using our custom classifier
        outputs = []
        for text in text_inputs:
            # Get prediction from our custom sentiment classifier
            result = predict_sentiment(text)

            # Convert prediction to probability format expected by TextAttack
            # TextAttack expects [negative_prob, positive_prob]
            negative_prob = result['probabilities']['negative']
            positive_prob = result['probabilities']['positive']
            probs = [negative_prob, positive_prob]

            outputs.append(probs)

        # Return tensor of predictions
        return torch.tensor(outputs)
```

```bash
# Import required libraries
import torch
from textattack.attack_recipes import TextFoolerJin2019
from textattack import Attacker, AttackArgs
from textattack.datasets import Dataset
from textattack.attack_results import SuccessfulAttackResult, FailedAttackResult
from textattack_wrapper import SentimentWrapper
from sentiment_classifier import predict_sentiment

# Download the NLTK library files to run the program
import nltk
nltk.download('averaged_perceptron_tagger_eng')

# Initialize the model wrapper
print("Initializing model wrapper...")
model_wrapper = SentimentWrapper()

# Create a dataset of examples to attack
# Format: (text, label) where label is 0 for negative, 1 for positive
examples = [
    ("This movie is great and amazing!", 1),
    ("This was a terrible waste of time.", 0),
    ("I really enjoyed watching this film.", 1),
    ("The worst movie I've ever seen.", 0),
    ("The acting was superb and the storyline kept me engaged throughout.", 1),
    ("I found the plot confusing and the characters were poorly developed.", 0),
    ("This is one of the best films I have watched in recent years.", 1),
    ("The movie was disappointing and failed to meet my expectations.", 0),
    ("I absolutely loved the cinematography and the musical score was fantastic.", 1),
    ("The film was boring and I struggled to stay awake during the entire screening.", 0),
    ("The movie was quite good and I found it entertaining.", 1),
    ("I think this product is nice and works well for my needs.", 1),
    ("The service was acceptable and the staff were helpful.", 1),
    ("This book is interesting and kept my attention throughout.", 1),
    ("The food was decent and the prices seemed reasonable.", 1),
    # Negative examples (that could be attacked while preserving meaning)
    ("The movie was somewhat disappointing and felt a bit slow.", 0),
    ("I found the product mediocre and it didn't meet my expectations.", 0),
    ("The service was okay but nothing special and rather basic.", 0),
    ("The book was average and I thought it was somewhat boring.", 0),
    ("The food was passable but the experience left me unimpressed.", 0)
]

# Convert to TextAttack dataset format
dataset = Dataset(examples)

# Create the TextFooler attack
print("Creating attack...")
attack = TextFoolerJin2019.build(model_wrapper)

# Set up attack arguments
print("Setting up attack arguments...")
attack_args = AttackArgs(
    num_examples=20,  # Number of examples to attack
    log_to_csv="attack_results.csv",  # Save results to CSV file
    checkpoint_interval=5,  # Save a checkpoint every 5 examples
    disable_stdout=False  # Show output in terminal
)

# Start the attack
print("Starting attack...")
attacker = Attacker(attack, dataset, attack_args)
results = attacker.attack_dataset()

# Custom analysis of results
print("\nCustom Attack Results Summary:")
print("-" * 50)

# Initialize counters
successful = 0
failed = 0
total_words_changed = 0
total_words = 0

# Detailed analysis of each result
print("\nDetailed Results:")
print("=" * 80)
for i, result in enumerate(results, 1):
    print(f"\nExample {i}:")
    print("-" * 80)

    # Get the original text
    original_text = str(result.original_result.attacked_text)

    # Get original sentiment prediction
    original_sentiment = predict_sentiment(original_text)
    original_label = original_sentiment['label']
    original_conf = original_sentiment['confidence']

    print(f"Original text: {original_text}")
    print(f"Original sentiment: {original_label} (confidence: {original_conf:.4f})")

    # Check if attack was successful
    if isinstance(result, SuccessfulAttackResult):
        # Get the perturbed text
        perturbed_text = str(result.perturbed_result.attacked_text)

        # Get perturbed sentiment prediction
        perturbed_sentiment = predict_sentiment(perturbed_text)
        perturbed_label = perturbed_sentiment['label']
        perturbed_conf = perturbed_sentiment['confidence']

        print(f"\nPerturbed text: {perturbed_text}")
        print(f"Perturbed sentiment: {perturbed_label} (confidence: {perturbed_conf:.4f})")

        # Check if sentiment actually flipped
        sentiment_flipped = (original_label != perturbed_label)
        if sentiment_flipped:
            print(f"SUCCESS: Sentiment flipped from {original_label} to {perturbed_label}!")
        else:
            print(f"WARNING: Attack marked as successful but sentiment did not flip!")

        # Calculate word changes
        orig_words = original_text.split()
        pert_words = perturbed_text.split()

        # Simple word change calculation
        words_changed = sum(1 for o, p in zip(orig_words, pert_words) if o != p)
        if len(orig_words) != len(pert_words):
            words_changed += abs(len(orig_words) - len(pert_words))

        # Update counters
        successful += 1
        total_words_changed += words_changed
        total_words += len(orig_words)

        # Print statistics
        print(f"\nWords changed: {words_changed}")
        print(f"Modification rate: {words_changed/len(orig_words):.2%}")
    else:
        print(f"\n✗ Attack failed - Could not change sentiment from {original_label}")
        failed += 1

    print("-" * 80)

# Print summary statistics
print("\nSummary Statistics:")
print("-" * 50)
print(f"Number of successful attacks: {successful}")
print(f"Number of failed attacks: {failed}")
print(f"Success rate: {(successful/len(results)):.2%}")

if successful > 0:
    print(f"Average word modification rate: {(total_words_changed/total_words):.2%}")

# Save results to file
with open("attack_summary.txt", "w") as f:
    f.write("Attack Results Summary\n")
    f.write("-" * 50 + "\n")
    f.write(f"Number of successful attacks: {successful}\n")
    f.write(f"Number of failed attacks: {failed}\n")
    f.write(f"Success rate: {(successful/len(results)):.2%}\n")
    if successful > 0:
        f.write(f"Average word modification rate: {(total_words_changed/total_words):.2%}\n")

    # Write detailed results
    f.write("\nDetailed Results:\n")
    f.write("-" * 50 + "\n")
    for i, result in enumerate(results, 1):
        original_text = str(result.original_result.attacked_text)
        original_sentiment = predict_sentiment(original_text)
        f.write(f"\nExample {i}:\n")
        f.write(f"Original: {original_text}\n")
        f.write(f"Original sentiment: {original_sentiment['label']} ({original_sentiment['confidence']:.4f})\n")

        if isinstance(result, SuccessfulAttackResult):
            perturbed_text = str(result.perturbed_result.attacked_text)
            perturbed_sentiment = predict_sentiment(perturbed_text)
            f.write(f"Perturbed: {perturbed_text}\n")
            f.write(f"Perturbed sentiment: {perturbed_sentiment['label']} ({perturbed_sentiment['confidence']:.4f})\n")
            if original_sentiment['label'] != perturbed_sentiment['label']:
                f.write("Status: SUCCESS - Sentiment flipped!\n")
            else:
                f.write("Status: WARNING - Attack successful but sentiment did not flip\n")
        else:
            f.write("Status: FAILED\n")
        f.write("-" * 50 + "\n")

print("\nResults saved to attack_summary.txt")
```
