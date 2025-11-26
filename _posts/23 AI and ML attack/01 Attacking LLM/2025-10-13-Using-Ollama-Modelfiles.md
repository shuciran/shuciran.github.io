---
description: >-
  Using Ollama Modelfiles
title: Using Ollama Modelfiles     # Add title here
date: 2025-10-13 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 01 Attacking LLM]                     # Change Templates to Writeup
tags: [AI, ollama, ML, Modelfile ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Understanding Modelfiles

A basic Modelfile structure:

```bash
FROM <base-model>                    # Base model to customize
PARAMETER <key> <value>              # Model parameters
SYSTEM <system-prompt>               # System-level instructions
TEMPLATE <template> 
```
Creating a Cybersecurity Expert Model
```bash
cat > modelfile-vuln-expert << 'EOF'
FROM mistral
PARAMETER temperature 0.3
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1
SYSTEM You are an expert vulnerability researcher and penetration tester with 15+ years of experience. You specialize in finding and analyzing security vulnerabilities, writing technical security reports, and providing step-by-step exploitation guides. Always format responses with clear headings, include severity ratings (Critical/High/Medium/Low), and emphasize responsible disclosure practices.
EOF
```
Create the specialized model:
```bash
ollama create vuln-expert -f modelfile-vuln-expert
```
Test the vulnerability expert:
```bash
ollama run vuln-expert "How would you test a web application for SQL injection vulnerabilities?"
```
#### Parameter Optimization for Security Tasks

High Accuracy Tasks (Compliance, Analysis)
```bash
PARAMETER temperature 0.1-0.3      # Low randomness for consistency
PARAMETER top_p 0.8-0.9           # Focus on most probable responses
PARAMETER repeat_penalty 1.1-1.2  # Avoid repetitive analysis
```
> Why these settings? Balanced temperature provides quick but accurate responses for time-sensitive security alerts. Limited prediction length and reduced top_k ensure faster response times when speed is critical.
{: .prompt-info }

#### Creating Additional Specialized Models

Incident Response Specialist

```bash
cat > modelfile-incident-response << 'EOF'
FROM mistral
PARAMETER temperature 0.2
PARAMETER num_ctx 4096
SYSTEM You are a senior incident response analyst and digital forensics expert. Your expertise includes rapid incident triage, digital evidence collection, malware analysis, and creating detailed incident timelines. Always provide structured, actionable responses with clear priorities using established frameworks like NIST or SANS.
EOF
```
#### Secure Code Review Assistant

```bash
cat > modelfile-code-security << 'EOF'
FROM codellama:7b
PARAMETER temperature 0.1
PARAMETER num_ctx 8192
SYSTEM You are an expert secure code reviewer with deep knowledge of OWASP Top 10, CWE, and language-specific security issues. When reviewing code, identify specific vulnerabilities with line references, explain security impact, provide secure code examples, and rate severity of findings.
EOF
```
Create these models:
```bash
ollama create incident-response -f modelfile-incident-response
```

```bash
ollama create code-security -f modelfile-code-security
```
Let’s run these models:
```bash
ollama run incident-response "We detected suspicious network traffic from IP 192.168.1.100. What are the immediate response steps?"
```
And another one:
```bash
ollama run code-security "Review this SQL query: SELECT * FROM users WHERE id = '" + userId + "'"
```
