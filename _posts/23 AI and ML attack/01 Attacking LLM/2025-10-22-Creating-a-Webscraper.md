---
description: >-
  Creating a Webscraper using Pyscrap
title: Creating a Webscraper using Pyscrap        # Add title here
date: 2025-10-22 08:00:00 -0600                           # Change the date to match completion date
categories: [23 AI and ML attack, Attacking LLM]                     # Change Templates to Writeup
tags: [AI, Webscraper,Pyscrap]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Simple Scrapper

```bash
import requests
import sys
from bs4 import BeautifulSoup

def scrape(url, max_chars):

    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    # Find all tags that typically contain text (p, div, span, h1, h2, h3, h4, h5, h6, article, section, etc.)
    text_content = []
    for tag in soup.find_all(['p', 'div', 'span', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'article', 'section']):
        if tag.get_text(strip=True):
            text_content.append(tag.get_text(strip=True))

    # Combine all text from selected tags
    full_text = '\n'.join(text_content)

    # Return the required amount of text based on max_chars
    return(full_text[:max_chars])

if __name__ == '__main__':
    # Take URL and max characters as command-line arguments
    if len(sys.argv) != 3:
        print("Usage: python simple_scraper.py <URL> <max_characters>")
        sys.exit(1)

    url = sys.argv[1]
    if not url.startswith("http"):
        print("Error: Invalid URL")
        sys.exit(1)

    try:
        max_chars = int(sys.argv[2])
    except ValueError:
        print("Please enter a valid integer for max_characters.")
        sys.exit(1)

    web_content = scrape(url, max_chars)
    print(web_content)
```


```bash
wget -O simple_scraper.py https://gitlab.practical-devsecops.training/-/snippets/73/raw/main/simple_scraper.py
```