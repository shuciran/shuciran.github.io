---
description: >-
  Finding and Fixing Weaknesses in AI Code
title: Finding and Fixing Weaknesses in AI Code   # Add title here
date: 2025-11-27 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 03 Attacking AI using DevOps]                     # Change Templates to Writeup
tags: [AI, Weakness AI Code]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
Understanding Static Application Security Testing

SAST helps in:

1) Finding the weaknesses in the code that might materialize into vulnerabilities.
2) Providing recommendations for fixing the weaknesses.

#### Downloading the Source Code
```bash
git clone https://gitlab.practical-devsecops.training/marudhamaran/caisp-image-classifier.git && cd caisp-image-classifier
```

#### Installing Bandit

```bash
pip install bandit==1.8.3
```

Usage:
```bash
bandit -r .

[main]  INFO    profile include tests: None
[main]  INFO    profile exclude tests: None
[main]  INFO    cli include tests: None
[main]  INFO    cli exclude tests: None
[main]  INFO    running on Python 3.10.12
Run started:2025-11-28 05:01:02.086392

Test results:
>> Issue: [B614:pytorch_load] Use of unsafe PyTorch load
   Severity: Medium   Confidence: High
   CWE: CWE-502 (https://cwe.mitre.org/data/definitions/502.html)
   More Info: https://bandit.readthedocs.io/en/1.8.3/plugins/b614_pytorch_load.html
   Location: ./image-classifier.py:113:30
112             # load best model weights
113             model.load_state_dict(torch.load(best_model_params_path))
114         return model

--------------------------------------------------
>> Issue: [B614:pytorch_load] Use of unsafe PyTorch load
   Severity: Medium   Confidence: High
   CWE: CWE-502 (https://cwe.mitre.org/data/definitions/502.html)
   More Info: https://bandit.readthedocs.io/en/1.8.3/plugins/b614_pytorch_load.html
   Location: ./image-classifier.py:160:20
159             if torch.__version__[:3] == '2.3':
160                 model = torch.load(model_file)
161             else:

--------------------------------------------------
>> Issue: [B614:pytorch_load] Use of unsafe PyTorch load
   Severity: Medium   Confidence: High
   CWE: CWE-502 (https://cwe.mitre.org/data/definitions/502.html)
   More Info: https://bandit.readthedocs.io/en/1.8.3/plugins/b614_pytorch_load.html
   Location: ./image-classifier.py:175:24
174                 ):
175                     model = torch.load(model_file)
176         else:

--------------------------------------------------

Code scanned:
        Total lines of code: 140
        Total lines skipped (#nosec): 0

Run metrics:
        Total issues (by severity):
                Undefined: 0
                Low: 0
                Medium: 3
                High: 0
        Total issues (by confidence):
                Undefined: 0
                Low: 0
                Medium: 0
                High: 3
Files skipped (0):
```

Bandit’s results indicate that:

There are 3 weaknesses in the source code.
3 of the weaknesses are medium severity weaknesses.
Bandit is pretty confident about the results, indicating that those three weaknesses are reported with high confidence.

The three issues were reported in the following line numbers in the image-classifier.py file:

113
160
175
The code present around those lines were also shown, but we did not include them in the above output for brevity.

The three issues are related to the use of torch.load function.

The bandit tool has provided the following links to help us understand the issue, and how to fix it:

https://bandit.readthedocs.io/en/1.8.3/plugins/b614_pytorch_load.html
https://cwe.mitre.org/data/definitions/502.html
Let’s navigate to the first link, and understand the issue - https://bandit.readthedocs.io/en/1.8.3/plugins/b614_pytorch_load.html.

![](/assets/img/Pasted-image-20251127230517.png)

To fix, the documentation recommends the following:
```bash
 There are two safe alternatives:
 1. Use torch.load with weights_only=True where only tensor data is extracted, and no arbitrary Python objects are deserialized
 2. Use the safetensors library from huggingface, which provides a safe deserialization mechanism
```

With weights_only=True, PyTorch enforces a strict type check, ensuring that only torch.Tensor objects are loaded.
So, it appears that the torch.load function is not safe to use, the way it is currently used in the code.

And, it also appears that the torch.load function also takes another parameter called weights_only, which when set to True, ensures that only tensor data is loaded, and no arbitrary Python objects are deserialized.

Let’s go ahead and fix the issue by adding the weights_only parameter to the torch.load function.

You can open the image-classifier.py file in vi or nano editor, and add the weights_only parameter to the torch.load function, set to True.

Or you can use the below sed command to that in an easy way.

```bash
sed -i "s/torch.load(\(best_model_params_path\))/torch.load(\1, weights_only=True)/g" image-classifier.py
sed -i "s/torch.load(\(model_file\))/torch.load(\1, weights_only=True)/g" image-classifier.py
```