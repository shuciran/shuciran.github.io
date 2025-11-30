---
description: >-
  Threat Modeling an AI System
title: Threat Modeling an AI System   # Add title here
date: 2025-11-29 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, 04 Threat Modeling]                     # Change Templates to Writeup
tags: [AI, Picklescan]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

#### Introduction
Scope of threat modeling can be the amount of application features or design that needs to be threat modeled, and the time that is available for the threat modeling activity.

#### Traditional Threat Modeling Scope vs Agile Threat Modeling Scope

Scope is what actually differentiates traditional threat modeling and threat modeling in Agile environments.

In traditional threat modeling the scope of threat model is big - usually an entire system is threat modeled because of big releases.

Threat modeling activity is sometimes performed by expert threat modelers, might take weeks to produce outcome, and threats are maintained in documents.

#### The Interplay of Threats, and Requirements

Security requirements can come from compliance requirements.

Compliance requirements can be contractual, regulatory, or statutory.

We might have a compliance requirement that says the system should not have any buffer overflows, or injection attacks.

During threat modeling we would want to mitigate the threat of attacks on a database through SQL injection.

So, a security requirement can be identifying an input validation strategy and implementing the input validation strategy based on various different data types.

When performing threat modeling, the threats and the identified mitigations can also feed as a very useful input to security requirements.

On the other hand, when identifying mitigations for a threat, or ranking a threat, we might also want to consider that we already have a security requirement to implement input validation, and hence the likelihood of a particular threat materializing in to an attack could become lower, resulting in a lesser score for that particular threat.

#### Ideas to Identify Security Requirements
There are multiple ways to define security requirements for a product or even a single user story.

Some of the most common ways are provided below as an example:

1) Level 1 requirements in OWASP ASVS
2) All threats in Mitre CAPEC that are rated High should be addressed
3) STRIDE’s security properties should be implemented in a feature
4) The system should not contain OWASP Top 10 2021 list of risks
5) PCI DSS requirements number 6 “Develop and maintain secure systems and applications” should be satisfied
6) IEEE’s EAD 2.0 requirements should be satisfied
7) NIST’s AI Risk Management Framework should be satisfied

Alternatively, your organization might have its own list of security requirements for your applications to adapt.

For this exercise, since we are going to use STRIDE mnemonic to identify threats, we can simply state that the AI connected system should have the below security properties:

1) Authentication [Defense against Spoofing threats]
2) Integrity [Defense against Tampering threats]
3) Non-Repudiation [Defense against Repudiation threats]
4) Confidentiality [Defense against Information Disclosure threats]
5) Availability [Defense against Denial of Service threats]
6) Authorization [Defense against Privilege Escalation threats]

#### Diagramming for Threat Modeling

We are going to consider this architecture diagram for threat modeling the AI connected system.

https://app.diagrams.net/#Uhttps%3A%2F%2Fgitlab.practical-devsecops.training%2F-%2Fsnippets%2F78%2Fraw%2Fmain%2FAI-Training-Inference-Data-Architecture.drawio.DFDStencils.xml

#### The Architecture Diagram for AI

Below is an architecture diagram that represents the components that are involved in the AI connected system.

![](/assets/img/Pasted-image-20251129025749.png)

##### Architecture Components

###### External Entities
- User interacting with system through Browser/App
###### Network Zones
1) DMZ (Public-facing)
   - API Gateway/Proxy for request routing
   - Security Authentication System for auth & access control
2) Internal Network - Interface to DMZ
   - AI Model (Inference) for request processing
3) Internal Network - Protected
   - Data Layer for user data storage
   - Logs for system activity tracking
4) Internal Network - Development
- Training Data storage
- Model Training Pipeline for model improvements
###### Data Flows
a) User Request Flow
  1) User → Browser/App → API Gateway/Proxy
  2) API Gateway/Proxy ↔ Security Authentication System
  3) Security Authentication System ↔ AI Model
  4) AI Model → Data Layer
  5) AI Model → Logs
b) Training Flow
  1) Training Data → Model Training Pipeline
  2) Model Training Pipeline → AI Model
###### Existing Security Controls
- DMZ isolation for public-facing components
- Authentication and access control at gateway
- Protected internal network for sensitive data
- Separate development environment for training

