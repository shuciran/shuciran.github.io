---
description: >-
  Analyzing and Fixing Vulnerabilities in Third-Party Components
title: Analyzing and Fixing Vulnerabilities in Third-Party Components    # Add title here
date: 2025-11-26 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 03 Attacking AI using DevOps]                     # Change Templates to Writeup
tags: [AI, 3rd party vulns]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction

Software Component Analysis (SCA) helps in:

1) Identifying the components used in a software.
2) Understanding the dependencies between the components.
3) Understanding the potential vulnerabilities in the components.
4) Providing recommendations for fixing the vulnerabilities.

What is Needed for Software Component Analysis?
To perform software component analysis, we need the following:

1) The source code of the software.
2) The list of dependencies of the software.
3) The list of dependencies to be installed in the system, before starting an SCA scan.

Here’s an example of python’s requirements.txt file specifying a range of versions:
```python 
# Flask 2.2.0 or newer
Flask>=2.2.0

# Requests library version 2.25.0 or newer
requests>=2.25.0

# Numpy 1.21.x only (>=1.21.0 and <1.22.0) for compatibility with specific APIs
numpy~=1.21.0
```

```json
And, an example of a package.json file specifying a range of versions:

{
  "name": "example-node-project",
  "version": "1.0.0",
  "dependencies": {
    "express": ">=4.17.0", // Use Express 4.17.0 or newer
    "lodash": "^4.17.0",   // Accepts any version >= 4.17.0 and < 5.0.0 (compatible minor & patch)
    "axios": "~1.3.0"      // Accepts any version >= 1.3.0 and < 1.4.0 (patch updates only)
  }
}
```

#### Downloading the Source Code (Application that is used to classify images)
````bash
git clone https://gitlab.practical-devsecops.training/marudhamaran/caisp-image-classifier.git && cd caisp-image-classifier
````

##### The Dependency File
```bash
cat > requirements.txt << EOF
torch==2.3.0
torchvision==0.18.0
Pillow==10.0.0
EOF
```

#### Installing Grype
```bash
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
```

Usage:
```bash
grype .

 ✔ Indexed file system                                                                          . 
 ✔ Cataloged contents              cdb4ee2aea69cc6a83331bbe96dc2caa9a299d21329efb0336fc02a82e1839a  
   ├── ✔ Packages                        [3 packages]  
   ├── ✔ Executables                     [0 executables]  
   ├── ✔ File digests                    [1 files]  
   └── ✔ File metadata                   [1 locations]  
 ✔ Scanned for vulnerabilities     [1 vulnerability matches]  
   ├── by severity: 0 critical, 0 high, 1 medium, 0 low, 0 negligible
   └── by status:   0 fixed, 1 not-fixed, 0 ignored 
[0000]  WARN no explicit name and version provided for directory source, deriving artifact ID from t
NAME   INSTALLED  TYPE    VULNERABILITY        SEVERITY  EPSS %  RISK   
torch  2.7.1      python  GHSA-887c-mr87-cxwp  Medium    5.64    < 0.1
```

You need to run it on the cloned git 