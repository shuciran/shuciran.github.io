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

### Creating a Webscraper using Pyscrap

Setting Up Our Environment
Let’s start by updating our package lists and installing the necessary tools:

```bash
apt update
```
Now let’s install Python3, Python virtual environment support, and pip:
```bash
apt install -y python3 python3.10-venv python3-pip -y
```
Next, let’s install Scrapy, the Python framework we’ll use for web scraping:
```bash
pip install scrapy==2.12.0
```
Let’s create a new Scrapy project:
```bash
scrapy startproject webscraper
cd webscraper
```
To better visualize our project structure, let’s install the tree utility:
```bash
apt install -y tree
```
Scrapy requires spiders to be defined as Python classes. Let’s create a file named altoro_spider.py in the spiders directory:
```bash
cat > webscraper/spiders/altoro_spider.py <<EOF
import scrapy

class AltoroSpider(scrapy.Spider):
    name = "altoro"
    allowed_domains = ['demo.testfire.net']
    start_urls = ['https://demo.testfire.net/login.jsp']
EOF
```
Next, we’ll add the main parsing method that extracts data from the initial page:
```bash
cat >> webscraper/spiders/altoro_spider.py <<EOF
    def parse(self, response):
        # Extract main navigation links
        nav_links = response.css('a::attr(href)').getall()

        # Extract main sections
        sections = {
            'personal': response.xpath('//a[contains(@href, "personal")]'),
            'business': response.xpath('//a[contains(@href, "business")]'),
            'inside': response.xpath('//a[contains(@href, "inside")]')
        }
EOF
```
Next, we’ll add the method that handles the content pages:
```bash
cat >> webscraper/spiders/altoro_spider.py <<EOF

    def parse_content_page(self, response):
        yield {
            'url': response.url,
            'title': response.css('title::text').get(),
            'content': response.css('div.content::text').getall(),
            'links': response.css('a::attr(href)').getall()
        }
EOF
```
Finally, we’ll add a helper method to process section links:
```bash
cat >> webscraper/spiders/altoro_spider.py <<EOF

    def extract_section_links(self, section):
        return [{'text': link.css('::text').get(), 
                 'url': link.css('::attr(href)').get()} 
                for link in section]
EOF
```
Now let’s create a settings file to configure our scraper:
```bash
cat > webscraper/settings.py <<EOF
# Scrapy settings for webscraper project

BOT_NAME = "altoro_scraper"

SPIDER_MODULES = ["webscraper.spiders"]
NEWSPIDER_MODULE = "webscraper.spiders"

# Respect the site's robots.txt
ROBOTSTXT_OBEY = True

# Add significant delays between requests to avoid overloading the server
DOWNLOAD_DELAY = 3

# Identify your scraper (always use a descriptive user agent)
USER_AGENT = "AltoroScraperBot (+https://yourwebsite.com)"

# Don't overload the demo site with concurrent requests
CONCURRENT_REQUESTS = 1

# Cache responses to reduce server load
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 86400

# Set settings whose default value is deprecated to a future-proof value
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"
FEED_EXPORT_ENCODING = "utf-8"
EOF
```
Now we can run our web scraper to collect data:
```bash
cd ~
cd webscraper
scrapy crawl altoro -O altoro_structure.json
```
Once the crawl is complete, you can view the scraped data:
```bash
cat altoro_structure.json | jq
```
Let’s create our analysis script:
```bash
cat > analyze_scraped_data.py <<EOF
import json
from collections import Counter

# Load the scraped data
with open('altoro_structure.json', 'r') as f:
    data = json.load(f)
EOF
```
Add code to count unique URLs:
```bash
cat >> analyze_scraped_data.py <<EOF

# Count all unique URLs
all_urls = []
for item in data:
    # Add the main URL of each page
    if 'url' in item:
        all_urls.append(item['url'])
    # Add all links found on the page
    if 'links' in item:
        all_urls.extend(item['links'])

# Use a set to remove duplicates
unique_urls = set(all_urls)
print(f"Total unique URLs found: {len(unique_urls)}")
EOF
```
Now add code to categorize the content:
```bash
cat >> analyze_scraped_data.py <<EOF

# Analyze content by page title
content_types = Counter()
for item in data:
    if 'title' in item and isinstance(item.get('title'), str):
        title = item['title'].lower()
        # Categorize based on keywords in titles
        if 'login' in title:
            content_types['login'] += 1
        elif 'personal' in title:
            content_types['personal'] += 1
        elif 'business' in title:
            content_types['business'] += 1
        elif 'about' in title or 'inside' in title:
            content_types['about'] += 1
        else:
            content_types['other'] += 1

print("\nContent type distribution:")
for content_type, count in content_types.items():
    print(f"- {content_type}: {count}")
EOF
```
Finally, add code to extract financial terminology:
```bash
cat >> analyze_scraped_data.py <<EOF

# Define banking/financial terms to look for
financial_terms = set()
banking_terms = ['account', 'banking', 'credit', 'loan', 'mortgage', 
                'investment', 'transfer', 'deposit', 'withdrawal',
                'checking', 'savings', 'retirement', 'insurance',
                'lending', 'cards', 'financial', 'business', 'personal']

# Search URLs for terms
for item in data:
    if 'url' in item and isinstance(item.get('url'), str):
        url_text = item['url'].lower()
        for term in banking_terms:
            if term in url_text:
                financial_terms.add(term)
EOF
```
Extract Terms from Titles
```bash
cat >> analyze_scraped_data.py <<EOF

    # Search page titles for terms
    if 'title' in item and isinstance(item.get('title'), str):
        title_text = item['title'].lower()
        for term in banking_terms:
            if term in title_text:
                financial_terms.add(term)
EOF
```
Extract Terms from Content
```bash
cat >> analyze_scraped_data.py <<EOF

    # Search content (if available) for terms
    if 'content' in item and isinstance(item.get('content'), list):
        for content in item['content']:
            if isinstance(content, str):
                content_text = content.lower()
                for term in banking_terms:
                    if term in content_text:
                        financial_terms.add(term)
EOF
```
Extract Terms from Navigation Sections
```bash
cat >> analyze_scraped_data.py <<EOF

    # Search navigation sections
    if 'main_sections' in item and isinstance(item.get('main_sections'), dict):
        # Check section names (like "personal", "business")
        for section_name, links in item['main_sections'].items():
            section_text = section_name.lower()
            for term in banking_terms:
                if term in section_text:
                    financial_terms.add(term)

            # Check links within sections
            if isinstance(links, list):
                for link in links:
                    if isinstance(link, dict):
                        # Check link text
                        if 'text' in link and isinstance(link['text'], str):
                            link_text = link['text'].lower()
                            for term in banking_terms:
                                if term in link_text:
                                    financial_terms.add(term)
EOF
```
Print the Results
```bash
cat >> analyze_scraped_data.py <<EOF

# Display all financial terms found
print("\nFinancial terms found:")
if financial_terms:
    for term in sorted(financial_terms):
        print(f"- {term}")
else:
    print("No financial terms found.")
EOF
```