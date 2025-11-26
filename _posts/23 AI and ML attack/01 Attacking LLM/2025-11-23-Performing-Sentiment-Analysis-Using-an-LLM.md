---
description: >-
  Performing Sentiment Analysis Using an LLM
title: Performing Sentiment Analysis Using an LLM    # Add title here
date: 2025-11-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Sentiment Analysis]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### About Sentiment Analysis
Sentimental Analysis is a crucial task in natural language processing (NLP) that involves determining the emotional tone or polarity of a given text, classifying it as positive, negative, or neutral.

However, it faces challenges such as understanding context, detecting sarcasm, handling ambiguous language, and adapting to domain-specific vocabulary.

#### Setting up the Attack Lab
Requirements:
```bash
apt update
apt install -y python3
apt install -y python3.10-venv
mkdir sentiment_attack
cd sentiment_attack
python3 -m venv venv
source venv/bin/activate
pip install transformers==4.49.0 torch==2.6.0
```

#### Attack script
```python
cat > chatattack.py <<EOF
from transformers import pipeline
import time

class StudentChatbotAttack:
    def __init__(self):
        print("🤖 Initializing Student's Chatbot Attack Lab...")
        self.analyzer = pipeline('sentiment-analysis')

        # Common sentiment words for reference
        self.word_guide = {
            "Positive Words": [
                "good", "great", "excellent", "amazing", "wonderful",
                "love", "happy", "fantastic", "brilliant", "awesome"
            ],
            "Negative Words": [
                "bad", "terrible", "awful", "horrible", "poor",
                "hate", "sad", "disappointing", "useless", "worst"
            ],
            "Intensity Words": [
                "very", "really", "extremely", "absolutely", "totally",
                "completely", "utterly", "incredibly", "seriously", "deeply"
            ]
        }

    def analyze_message(self, text):
        """Analyze message sentiment"""
        try:
            result = self.analyzer(text)[0]
            print(f"\n📊 Analysis of: '{text}'")
            print(f"Sentiment: {result['label']}")
            print(f"Confidence: {result['score']:.2%}")
            return result
        except Exception as e:
            print(f"❌ Error analyzing text: {e}")
            return None

    def show_word_guide(self):
        """Display available words for attacks"""
        print("\n📚 Word Guide for Attacks:")
        print("=" * 50)
        for category, words in self.word_guide.items():
            print(f"\n{category}:")
            for i, word in enumerate(words, 1):
                print(f"{i}. {word}", end='  ')
            print("\n")

def run_attack_lab():
    print("""
    🎓 Welcome to the Student's Chatbot Attack Lab! 
    ============================================

    In this lab, you'll learn how to:
    1. Analyze message sentiment
    2. Create adversarial attacks
    3. Test the effectiveness of your attacks

    ⚠️ Educational Purpose Only ⚠️
    """)

    lab = StudentChatbotAttack()

    while True:
        print("\n" + "="*50)
        print("🔍 Main Menu:")
        print("1. Start New Attack Exercise")
        print("2. View Word Guide")
        print("3. Practice with Examples")
        print("4. Exit Lab")

        choice = input("\nSelect option (1-4): ")

        if choice == '1':
            print("\n🎯 Starting New Attack Exercise")
            print("=" * 50)

            # Get original message
            original_message = input("\n1️⃣ Enter the message you want to attack: ")
            print("\nAnalyzing original message...")
            original_result = lab.analyze_message(original_message)

            if not original_result:
                continue

            # Show word guide
            lab.show_word_guide()

            print("\n2️⃣ Now, try to create an attack by:")
            print("- Replacing positive/negative words")
            print("- Adding intensity words")
            print("- Changing the sentence structure")

            while True:
                print("\n🔄 Attack Options:")
                print("1. Try an attack")
                print("2. View word guide again")
                print("3. Return to main menu")

                attack_choice = input("\nSelect option (1-3): ")

                if attack_choice == '1':
                    attack_message = input("\n✍️ Enter your attack message: ")
                    attack_result = lab.analyze_message(attack_message)

                    if attack_result:
                        if attack_result['label'] != original_result['label']:
                            print("\n🎉 Attack Successful!")
                            print("You changed the sentiment!")
                            print("\nWhat worked in your attack:")
                            print("1. Original sentiment:", original_result['label'])
                            print("2. New sentiment:", attack_result['label'])
                            print("3. Confidence change:", 
                                  f"{original_result['score']:.2%} → {attack_result['score']:.2%}")
                        else:
                            print("\n❌ Attack Failed")
                            print("The sentiment didn't change. Try:")
                            print("- Using stronger opposite words")
                            print("- Adding intensity words")
                            print("- Restructuring the sentence")

                        retry = input("\nWant to try another attack? (y/n): ")
                        if retry.lower() != 'y':
                            break

                elif attack_choice == '2':
                    lab.show_word_guide()
                else:
                    break

        elif choice == '2':
            lab.show_word_guide()

        elif choice == '3':
            print("\n📝 Example Messages to Practice With:")
            examples = [
                "I love this amazing chatbot!",
                "The service is excellent.",
                "This product is terrible.",
                "I'm happy with the results."
            ]
            for i, example in enumerate(examples, 1):
                print(f"{i}. {example}")

            ex_choice = input("\nChoose an example to attack (1-4): ")
            if ex_choice in ['1', '2', '3', '4']:
                lab.analyze_message(examples[int(ex_choice)-1])
                print("\nNow try attacking this message!")
                lab.show_word_guide()
            else:
                print("❌ Invalid choice!")

        elif choice == '4':
            print("\n👋 Thanks for participating in the Chatbot Attack Lab!")
            break

        else:
            print("\n❌ Invalid choice. Please try again.")

if __name__ == "__main__":
    run_attack_lab()
EOF
```
Usage:
```bash
python chatattack.py
```