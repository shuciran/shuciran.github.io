---
description: >-
  Text Classification using TensorFlow
title: Text Classification using TensorFlow       # Add title here
date: 2025-10-23 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Tensorflow]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Text Classification
Text classification is the task of assigning a label or category to a piece of text, such as an email, document, or sentence. In Natural Language Processing, text classification leverages machine learning models to automatically categorize text content.
Steps:
1) Create a Sample Dataset prepare_dataset.py
2) Building the Text Classification Model build_model.py
3) Training the Model train_model.py
4) Evaluating the Model evaluate_model.py
5) Creating an Interactive Interface classify_text.py 
Requirements:
```bash
apt update && apt install python3 python3.10-venv python3-pip -y
mkdir text_classification_lab
cd text_classification_lab
python3 -m venv venv
source venv/bin/activate
cat > requirements.txt<<EOF
tensorflow==2.18.0
pandas==2.2.3
numpy==2.0.2
scikit-learn==1.6.1
matplotlib==3.10.0
EOF
pip install -r requirements.txt
mkdir -p data/{train,test} models results
```

#### Creating a Sample Dataset
Download DataSet
```bash
wget -O sentiment_dataset_1000.csv https://gitlab.practical-devsecops.training/-/snippets/77/raw
```

```python
import pandas as pd
import os
from sklearn.model_selection import train_test_split

# Ensure directories exist
os.makedirs('data/train', exist_ok=True)
os.makedirs('data/test', exist_ok=True)

# Load the dataset
df = pd.read_csv('sentiment_dataset_1000.csv')

# Split into train and test sets (80% train, 20% test)
train_df, test_df = train_test_split(df, test_size=0.2, stratify=df['label'], random_state=42)

# Save to CSV files
train_df.to_csv('data/train/train.csv', index=False)
test_df.to_csv('data/test/test.csv', index=False)

print("Dataset prepared successfully!")
print(f"Training samples: {len(train_df)}")
print(f"Test samples: {len(test_df)}")
```
Run python
```bash
python prepare_dataset.py
```

#### Building the Text Classification Model
```python
import tensorflow as tf
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import pickle
import os

# Load the training data
train_data = pd.read_csv('data/train/train.csv')
texts = train_data['text'].values
labels = train_data['label'].values
# Parameters for text processing
MAX_WORDS = 5000  # Maximum vocabulary size
MAX_LENGTH = 100  # Maximum sequence length

# Create and fit tokenizer
tokenizer = Tokenizer(num_words=MAX_WORDS)
tokenizer.fit_on_texts(texts)

# Convert texts to sequences and pad them
sequences = tokenizer.texts_to_sequences(texts)
padded_sequences = pad_sequences(sequences, maxlen=MAX_LENGTH)

# Check class balance
positive_samples = sum(labels)
negative_samples = len(labels) - positive_samples
print(f"Positive samples: {positive_samples}")
print(f"Negative samples: {negative_samples}")
# Create the model
model = tf.keras.Sequential([
    # Embedding layer
    tf.keras.layers.Embedding(MAX_WORDS, 64, input_length=MAX_LENGTH),

    # Bidirectional LSTM layer
    tf.keras.layers.Bidirectional(tf.keras.layers.LSTM(64, return_sequences=True)),

    # Global pooling
    tf.keras.layers.GlobalMaxPooling1D(),

    # Dense layers
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dropout(0.5),
    tf.keras.layers.Dense(32, activation='relu'),
    tf.keras.layers.Dropout(0.5),

    # Output layer
    tf.keras.layers.Dense(1, activation='sigmoid')
])
# Compile the model
model.compile(
    optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
    loss='binary_crossentropy',
    metrics=['accuracy']
)

# Build the model with input shape to see parameters
model.build(input_shape=(None, MAX_LENGTH))

# Display model summary
model.summary()

# Save the tokenizer for later use
os.makedirs('models', exist_ok=True)
with open('models/tokenizer.pkl', 'wb') as f:
    pickle.dump(tokenizer, f)

print("Model built successfully and tokenizer saved.")
```
Run python
```bash
python build_model.py
```
#### Training the Model

```python
import tensorflow as tf
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import pickle
import os

# Load the training data
train_data = pd.read_csv('data/train/train.csv')
texts = train_data['text'].values
labels = train_data['label'].values

# Load the tokenizer
with open('models/tokenizer.pkl', 'rb') as f:
    tokenizer = pickle.load(f)

# Parameters
MAX_LENGTH = 100
# Prepare the data
sequences = tokenizer.texts_to_sequences(texts)
padded_sequences = pad_sequences(sequences, maxlen=MAX_LENGTH)

# Define class weights to handle any imbalance
class_weight = {0: 1.0, 1: 1.0}  # Adjust if classes are imbalanced

# Create the model with the same architecture as in build_model.py
model = tf.keras.Sequential([
    # Embedding layer
    tf.keras.layers.Embedding(5000, 64, input_length=MAX_LENGTH),

    # Bidirectional LSTM layer
    tf.keras.layers.Bidirectional(tf.keras.layers.LSTM(64, return_sequences=True)),

    # Global pooling
    tf.keras.layers.GlobalMaxPooling1D(),

    # Dense layers
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dropout(0.5),
    tf.keras.layers.Dense(32, activation='relu'),
    tf.keras.layers.Dropout(0.5),

    # Output layer
    tf.keras.layers.Dense(1, activation='sigmoid')
])

# Compile the model
model.compile(
    optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
    loss='binary_crossentropy',
    metrics=['accuracy']
)
# Add callbacks for better training
early_stopping = tf.keras.callbacks.EarlyStopping(
    monitor='val_loss',
    patience=3,
    restore_best_weights=True
)

reduce_lr = tf.keras.callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,
    patience=2
)

# Train the model
history = model.fit(
    padded_sequences,
    labels,
    epochs=10,
    batch_size=32,
    validation_split=0.1,
    callbacks=[early_stopping, reduce_lr],
    class_weight=class_weight,
    verbose=1
)

# Save the trained model
os.makedirs('models', exist_ok=True)
model.save('models/sentiment_model.keras')
# Plot training & validation accuracy values
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(history.history['accuracy'])
plt.plot(history.history['val_accuracy'])
plt.title('Model accuracy')
plt.ylabel('Accuracy')
plt.xlabel('Epoch')
plt.legend(['Train', 'Validation'], loc='upper left')

# Plot training & validation loss values
plt.subplot(1, 2, 2)
plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('Model loss')
plt.ylabel('Loss')
plt.xlabel('Epoch')
plt.legend(['Train', 'Validation'], loc='upper left')

# Save the plot
os.makedirs('results', exist_ok=True)
plt.tight_layout()
plt.savefig('results/training_history.png')
plt.close()

print("Model trained and saved successfully.")
print("Training visualization saved to results/training_history.png")
```

Run python
```bash
python train_model.py
```

#### Evaluating the Model
```python
import tensorflow as tf
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import pickle
import os
from sklearn.metrics import confusion_matrix, classification_report, ConfusionMatrixDisplay

# Load the test data
test_data = pd.read_csv('data/test/test.csv')
texts = test_data['text'].values
labels = test_data['label'].values

# Load the tokenizer
with open('models/tokenizer.pkl', 'rb') as f:
    tokenizer = pickle.load(f)

# Load the model
model = tf.keras.models.load_model('models/sentiment_model.keras')

# Parameters
MAX_LENGTH = 100
# Prepare the data
sequences = tokenizer.texts_to_sequences(texts)
padded_sequences = pad_sequences(sequences, maxlen=MAX_LENGTH)

# Evaluate the model
loss, accuracy = model.evaluate(padded_sequences, labels)
print(f"Test accuracy: {accuracy:.4f}")

# Make predictions
predictions = model.predict(padded_sequences)
predicted_labels = [1 if p > 0.5 else 0 for p in predictions]

# Create confusion matrix
cm = confusion_matrix(labels, predicted_labels)
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=["Negative", "Positive"])
disp.plot(cmap=plt.cm.Blues)
plt.title('Confusion Matrix')
plt.savefig('results/confusion_matrix.png')
plt.close()

# Classification report
report = classification_report(labels, predicted_labels, target_names=["Negative", "Positive"])
print("\nClassification Report:")
print(report)
# Print test data with predictions
print("\nDetailed Test Evaluation (Sample):")
print("-" * 80)

# Take a sample of 10 examples for detailed analysis
np.random.seed(42)
sample_indices = np.random.choice(len(texts), min(10, len(texts)), replace=False)

for idx in sample_indices:
    text = texts[idx]
    label = labels[idx]
    prediction = predictions[idx][0]
    predicted_label = 1 if prediction > 0.5 else 0
    correct = "✓" if predicted_label == label else "✗"
    sentiment = "Positive" if predicted_label == 1 else "Negative"
    true_sentiment = "Positive" if label == 1 else "Negative"

    print(f"Text: {text}")
    print(f"True label: {true_sentiment} | Predicted: {sentiment} (confidence: {prediction:.4f}) {correct}")
    print("-" * 80)
# Analyze prediction probabilities
plt.figure(figsize=(10, 6))
plt.hist([p[0] for p in predictions], bins=20, alpha=0.7)
plt.axvline(x=0.5, color='red', linestyle='--')
plt.title('Distribution of Prediction Probabilities')
plt.xlabel('Prediction Probability')
plt.ylabel('Frequency')
plt.savefig('results/prediction_distribution.png')
plt.close()

# Save evaluation results
results = {
    'text': texts,
    'true_label': labels,
    'predicted_probability': [p[0] for p in predictions],
    'predicted_label': predicted_labels,
    'correct': [pl == tl for pl, tl in zip(predicted_labels, labels)]
}

results_df = pd.DataFrame(results)
os.makedirs('results', exist_ok=True)
results_df.to_csv('results/evaluation_results.csv', index=False)

# Accuracy
accuracy = sum(results['correct']) / len(results['correct'])
print(f"\nOverall Accuracy: {accuracy:.4f}")
print("Evaluation completed and results saved.")
```

Run python
```bash
python evaulate_model.py
```

#### Creating an Interactive Interface
```python
import tensorflow as tf
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
import pickle
import os

# Load the tokenizer
with open('models/tokenizer.pkl', 'rb') as f:
    tokenizer = pickle.load(f)

# Load the model
model = tf.keras.models.load_model('models/sentiment_model.keras')

# Parameters
MAX_LENGTH = 100
# Function to classify text
def classify_text(text):
    # Tokenize and pad the text
    sequences = tokenizer.texts_to_sequences([text])
    padded = pad_sequences(sequences, maxlen=MAX_LENGTH)

    # Make prediction
    prediction = model.predict(padded)[0][0]

    # Determine sentiment and confidence
    if prediction > 0.5:
        sentiment = "POSITIVE"
        confidence = prediction
    else:
        sentiment = "NEGATIVE"
        confidence = 1 - prediction

    return sentiment, confidence
# Interactive interface
print("\nText Classification System")
print("=" * 30)
print("Type 'quit' to exit")
print("Type 'file' to classify text from a file")

while True:
    print("\nEnter text to classify:")
    user_input = input("> ").strip()

    if user_input.lower() == 'quit':
        print("Goodbye!")
        break
    elif user_input.lower() == 'file':
        file_path = input("Enter file path: ").strip()
        try:
            with open(file_path, 'r') as file:
                text = file.read()
                sentiment, confidence = classify_text(text)
                print(f"\nClassification Results:")
                print(f"Text: {text[:100]}..." if len(text) > 100 else f"Text: {text}")
                print(f"Sentiment: {sentiment}")
                print(f"Confidence: {confidence:.4f}")
        except FileNotFoundError:
            print(f"Error: File '{file_path}' not found")
    elif user_input:
        # Classify the input text
        sentiment, confidence = classify_text(user_input)
        print(f"\nClassification Results:")
        print(f"Sentiment: {sentiment}")
        print(f"Confidence: {confidence:.4f}")
```

Run python
```bash
python classify_text.py
```

Usage Examples:
```bash
This product is amazing and works perfectly!

