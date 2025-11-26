---
description: >-
    Evaluating AI Models Using pytest-evals
title: Evaluating AI Models Using pytest-evals      # Add title here
date: 2025-10-22 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, pytest-evals]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### pytest-evals
Powerful testing framework designed specifically for AI model evaluation. We’ll implement a mock LLM system to enable testing without requiring external API keys.
As AI models become integral to production systems, proper evaluation is crucial for ensuring these models meet quality standards before deployment. Evaluation helps you:

- Identify model strengths and weaknesses
- Track performance over time
- Establish quality gates for deployment
- Compare different model versions
- Create reproducible test suites for regression testing

#### Requirements:
```bash
apt update && apt install python3-pip python3 python3.10-venv -y
mkdir llm-evaluation
cd llm-evaluation
python3 -m venv venv
source venv/bin/activate
pip install pytest==7.4.0 pandas==2.0.3 matplotlib==3.7.2 numpy==1.24.3 pytest-evals==0.3.4
mkdir -p mock_llm
mkdir -p tests/data
touch tests/__init__.py
touch tests/conftest.py

pytest --help
```
#### Creating a Mock LLM
```bash
mkdir -p mock_llm tests
touch mock_llm/__init__.py
touch tests/__init__.py
cat > mock_llm/simple_llm.py <<'EOF'
import random

class MockLLM:
    def __init__(self, quality_level=0.8):
        # Quality level affects how 'good' the model is (0-1)
        self.quality_level = quality_level

        # Response templates by category
        self.response_templates = {
            "account_access": [
                "You can access your account by clicking the profile icon.",
                "To reset your password, visit the forgot password page."
            ],
            "billing": [
                "Our refund policy allows returns within 30 days.",
                "Payment processing typically takes 2-3 business days."
            ],
            "general_inquiry": [
                "Our business hours are 9am-5pm, Monday through Friday.",
                "You can reach customer service at support@example.com."
            ]
        }

    def classify_text(self, text):
        # Simple keyword-based classification
        text = text.lower()
        if "account" in text or "password" in text or "login" in text:
            return "account_access"
        elif "payment" in text or "refund" in text or "charge" in text:
            return "billing"
        else:
            return "general_inquiry"
EOF
```
#### Basic Testing
```bash
cat > test_llm.py <<'EOF'
from mock_llm.simple_llm import MockLLM

# Create the LLM
llm = MockLLM()

# Test some examples
examples = [
    "How do I reset my password?",
    "When will my payment be processed?",
    "What are your opening hours?"
]

# Run classification
for example in examples:
    category = llm.classify_text(example)
    print(f"Text: '{example}'")
    print(f"Classification: '{category}'")
    print("---")
EOF
python test_llm.py
```
#### Creating pytest Configuration
This configuration adds our mock_llm module to the Python path and creates a pytest fixture that shares the classifier function across all tests.
```bash
cat > tests/conftest.py <<'EOF'
import pytest
import sys
import os

# Add the parent directory to path for imports
sys.path.append(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

# Import our LLM
from mock_llm.simple_llm import MockLLM

# Create a fixture for our classifier
@pytest.fixture
def classifier():
    llm = MockLLM()
    return llm.classify_text
EOF
cat > tests/test_simple_eval.py <<'EOF'
import pytest

@pytest.mark.eval(name="my_classifier")
def test_customer_inquiry(eval_bag, classifier):
    # Input text to classify
    input_text = "I need to change my account password"

    # Get prediction using our classifier fixture
    prediction = classifier(input_text)

    # Store results in eval_bag
    eval_bag.input = input_text
    eval_bag.prediction = prediction

    # Incorrectly expect "billing" instead of "account_access"
    eval_bag.expected = "billing"  # This is wrong!

    # Calculate accuracy (1 if correct, 0 if incorrect)
    eval_bag.accuracy = 1 if prediction == eval_bag.expected else 0
    eval_bag.confidence = 0.95
EOF
```
Running the evaluation phase:
```bash
pytest --run-eval -v
```
### Creating an Analysis Test
```bash
cat > tests/test_analysis.py <<'EOF'
import pytest

@pytest.mark.eval_analysis(name="my_classifier")
def test_analysis(eval_results):
    # Count total test cases
    total_cases = len(eval_results)

    # Skip if no results found
    if total_cases == 0:
        pytest.skip("No evaluation results found. Run with --run-eval first.")

    # Count correct predictions
    correct_predictions = sum(1 for result in eval_results 
                            if hasattr(result, 'result') 
                            and 'accuracy' in result.result 
                            and result.result['accuracy'])

    # Calculate overall accuracy
    accuracy = correct_predictions / total_cases

    # Print results
    print(f"Total test cases: {total_cases}")
    print(f"Correct predictions: {correct_predictions}")
    print(f"Accuracy: {accuracy:.2%}")

    # Assert minimum quality threshold
    assert accuracy >= 0.7, f"Accuracy {accuracy:.2%} below threshold of 70%"
EOF
pytest --run-eval-analysis -v
```
#### Fixing the Evaluation Test
```bash
cat > tests/test_simple_eval.py <<'EOF'
import pytest

@pytest.mark.eval(name="my_classifier")
def test_customer_inquiry(eval_bag, classifier):
    # Input text to classify
    input_text = "I need to change my account password"

    # Get prediction using our classifier fixture
    prediction = classifier(input_text)

    # Store results in eval_bag
    eval_bag.input = input_text
    eval_bag.prediction = prediction

    # Use the correct expectation
    eval_bag.expected = "account_access"  # Fixed!

    # Calculate accuracy (1 if correct, 0 if incorrect)
    eval_bag.accuracy = 1 if prediction == eval_bag.expected else 0
    eval_bag.confidence = 0.95
EOF
pytest --run-eval --run-eval-analysis -v
```
#### Adding More Test Cases
```bash
cat > tests/test_multiple_cases.py <<'EOF'
import pytest

@pytest.mark.parametrize("input_text,expected", [
    ("I need to reset my password", "account_access"),
    ("How do I update my account details?", "account_access"),
    ("When will my payment be processed?", "billing"),
    ("What's your refund policy?", "billing"),
    ("What are your hours of operation?", "general_inquiry"),
])
@pytest.mark.eval(name="my_classifier")
def test_classifier_cases(eval_bag, classifier, input_text, expected):
    # Get prediction
    prediction = classifier(input_text)

    # Store results
    eval_bag.input = input_text
    eval_bag.prediction = prediction
    eval_bag.expected = expected
    eval_bag.accuracy = 1 if prediction == expected else 0
EOF
pytest --run-eval --run-eval-analysis -v
```

#### Key Benefits of pytest-evals for LLM Testing
Here’s why pytest-evals is great for testing AI models:

1) Two Simple Steps:
- First, we collect what the AI says (using –run-eval)
- Then, we check if it’s right (using –run-eval-analysis)
2) Easy Problem-Finding:
- We can see exactly which questions the AI got wrong
- We can tell if the AI is wrong or if our test is wrong
3) Better Testing:
- We look at how well the AI does overall, not just one answer
- We can check many different things about the AI’s answers
- We decide what score is good enough for our AI
4) Works for Big Tests:
- We can ask the AI hundreds of questions at once
- We can group similar questions together
- We can make different kinds of tests for the same AI
