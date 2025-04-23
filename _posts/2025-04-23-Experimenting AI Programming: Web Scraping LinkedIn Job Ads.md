---
title: "Experimenting AI programming: web scraping LinkedIn job ads"
permalink: "/posts/Experimenting-AI-programming-web-scraping-LinkedIn-job-ads"
header:
  teaser: /assets/images/web_scraping_linkedin_ads.png
excerpt: "AI programming"
date: April 23, 2025
show_date: true
toc: true
toc_sticky: true
toc_label: "Content"
comments: false
related: true
tags:
  - ChatGPT
  - Web scraping
  - Python
gallery1:
  - url: /assets/images/web_scraping_linkedin_ads.png
    image_path: /assets/images/web_scraping_linkedin_ads.png
    alt: "Snapshot of web scraping"  
gallery2:
  - url: /assets/images/web_scraping_linkedin_ads.gif
    image_path: /assets/images/web_scraping_linkedin_ads.gif
    alt: "Snapshot of web scraping recorded"
---

## Introduction

In this blog post, I’ll walk you through how I used **ChatGPT** and **Selenium** to build a simple yet powerful web scraping tool that helps backup LinkedIn job ads. This project was designed to automatically capture and store job details, allowing me to easily reference them later if the job ad becomes unavailable.

{% include gallery id="gallery1" caption=""%}

## Purpose of the Project

The primary goal of this project was to **automatically backup job ads** from LinkedIn after applying for a position, so they could be stored and reviewed later. Often, LinkedIn job postings are taken down after a job position is filled or closed, which can leave applicants without access to the original job description when preparing for interviews.

To give a clearer idea of how the code functions, here's an overview of the working flow:

1. **Read Job URLs from Excel**
The script begins by loading an Excel file that contains a list of job postings you have applied to. This Excel file should include a column with LinkedIn job URLs. We use the `openpyxl` library to read these URLs into a DataFrame.
2. **Launch Chrome with existing profile**
To avoid repeated login and verification issues, we launch Chrome manually using our existing profile where we're already logged into LinkedIn. This allows Selenium to control the browser without triggering login prompts or CAPTCHA.

3. **Open Each Job Link**
The script iterates through each job URL and opens it in the Chrome browser using Selenium’s WebDriver. A short delay is added to ensure the page loads completely.

4. **Expand the Job Description**
Some LinkedIn job postings have truncated descriptions that require clicking a "See More" button. The script detects and clicks this button if it's present to ensure the full job description is visible.

5. **Scroll and Capture the Screen**
The script scrolls slightly to adjust the job description view (to avoid sticky headers covering the content), then takes a screenshot of the job details section. This screenshot is saved with a filename like `job_01.png` in a predefined folder.

6. **Error Handling and Skipping Broken Links**
If a link is broken or an element isn’t found, the script logs an error message and moves on to the next job URL. This makes the script resilient and able to run over long job lists without breaking.

This workflow ensures that every job application you’ve made is backed up with a visual snapshot, which you can refer to during interview prep—even if the original posting disappears from LinkedIn.

{% include gallery id="gallery2" caption="Screen recording during the web scraping" %}


## Step 1: Setting Up the Environment

Before we begin, you'll need a couple of things:
- **Python** installed on your machine (ensure you have version 3.x).
- **Selenium** WebDriver for automating browser interactions.
- **ChromeDriver** to control Chrome from Selenium.
- **openpyxl** to read data from Excel (if you already have job URLs stored in an Excel sheet).

You can install the necessary libraries using `pip`:
```bash
pip install selenium openpyxl
```

We’ll also use **Google Chrome** for this task, but you can use any browser if you prefer.

## Step 2: The Scraping Script

With our environment ready, it’s time to write the code. The idea is to load job URLs from an Excel file and scrape the relevant details from LinkedIn job postings. Here's the basic structure of the code:

<details>
<summary>Python code</summary>
{% highlight python %}  
{% raw %}
import time
import os
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from openpyxl import load_workbook

# Load Excel file containing LinkedIn job URLs
file_path = "/path/to/job_ads.xlsx"
df = pd.read_excel(file_path, skiprows=0, engine="openpyxl")

# Get job URLs from Excel sheet
job_links = df["Job URL"].tolist()

# Define a folder to save screenshots
screenshot_folder = "/path/to/save/screenshots"
os.makedirs(screenshot_folder, exist_ok=True)

## Using Chrome profile for LinkedIn login
# After see the login, do it manually for the first time using the code
# Chrome must not be running!

# Path to your Chrome user data directory (replace this with your path)
user_data_dir = "your Chrome user data directory/Google/Chrome"

# Path to the profile you want to use (this is usually in the 'Profile 1', 'Profile 2', etc. directories)
profile_dir = "Default"  # Change to the actual profile you're using

# Setup Chrome options
options = Options()
options.add_argument("--start-maximized")
options.add_argument(f"user-data-dir={user_data_dir}")
options.add_argument(f"profile-directory={profile_dir}")  # Load your LinkedIn login profile

# Just use webdriver.Chrome() without specifying chromedriver path
driver = webdriver.Chrome(options=options)

# Open LinkedIn (already logged in if using the default profile)
driver.get("https://www.linkedin.com") # Then login manually for the first time using the code

# Iterate through job URLs and scrape details
for index, job_url in enumerate(job_links):
    if not job_url:
        print(f"Skipping row {index + 1}: No valid LinkedIn job URL.")
        continue

    try:
        driver.get(job_url)
        time.sleep(3)  # Wait for the page to load

        # Find the job description section
        job_desc = driver.find_element(By.CLASS_NAME, "job-view-layout")
        
        # Take a screenshot of the job details
        job_desc = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CLASS_NAME, "jobs-details")))
        job_desc.screenshot(screenshot_path)
        print(f"Saved screenshot for job ad {index + 1}: {screenshot_path}")

    except Exception as e:
        print(f"Row {index + 1}: Error opening job URL {job_url}. It may be removed. Error: {e}")
        continue

# Close the WebDriver
driver.quit()
{% endraw %}
{% endhighlight %}
</details>

## Step 3: Improving the Script with ChatGPT

While this basic script works, it could be improved in various ways. That's where **ChatGPT** came in to help me refine the code and fix some common issues I faced:

#### **1. Handling "See More" Buttons**

Sometimes, LinkedIn job descriptions are truncated and require clicking a "See More" button to view the full content. Initially, my script failed to handle this dynamic behavior. ChatGPT suggested using **WebDriverWait** to wait for the "See More" button to become clickable and then click it:

<details>
<summary>Python code</summary>
{% highlight python %}  
{% raw %}
# Click "See more" if available to expand job description
try:
    # Wait until the "See more" button is clickable
    see_more_button = WebDriverWait(driver, 5).until(
        EC.element_to_be_clickable((By.XPATH, "//button[@aria-label='Click to see more description']"))
    )
    
    # Try normal click
    see_more_button.click()
    time.sleep(2)
    print("Clicked 'See more' successfully!")

except Exception:
    try:
        # Try JavaScript click if normal click fails
        driver.execute_script("arguments[0].click();", see_more_button)
        print("Clicked 'See more' using JavaScript!")
        time.sleep(2)
    except Exception as e:
        print("No 'See more' button found or already expanded:", e)

time.sleep(3)
{% endraw %}
{% endhighlight %}
</details>

#### **2. Scrolling the Page**

To make sure I capture the entire job description, I needed to scroll the page to bring the "About the job" section to the top. ChatGPT guided me on using JavaScript commands to scroll down a few pixels, ensuring the full content was loaded before taking the screenshot:

<details>
<summary>Python code</summary>
{% highlight python %}  
{% raw %}
# Find all h2 elements for the "About the job" section
h2_elements = driver.find_elements(By.TAG_NAME, "h2")

# Locate the exact "About the job" section
about_section = None
for h2 in h2_elements:
    if "About the job" in h2.text:
        about_section = h2
        break

# If found, scroll to it
if about_section:
    driver.execute_script("arguments[0].scrollIntoView({block: 'start'});", about_section) # move to the top
    driver.execute_script("window.scrollBy(0, -130);")  # Move down a bit after scrolling
    print("Scrolled to 'About the job' section.")
    time.sleep(5)
else:
    print("'About the job' section not found.")
{% endraw %}
{% endhighlight %}
</details>

#### **3. Extracting the Correct Job Description**

ChatGPT also helped me fine-tune how to locate the exact job description section using the correct HTML tags. Initially, I was using a generic class name, but LinkedIn’s page structure was slightly more complex. By referencing specific div classes, ChatGPT helped me pinpoint the job description more accurately.

## Step 4: Optimizing for Robustness

One of the key learnings from ChatGPT was handling unexpected cases like missing job descriptions or incorrect links. I improved the script by adding more robust error handling:

<details>
<summary>Python code</summary>
{% highlight python %}  
{% raw %}
except Exception as e:
    print(f"Row {index + 1}: Error opening job URL {job_url}. It may be removed. Error: {e}")
    continue
{% endraw %}
{% endhighlight %}
</details>

This ensured that the script would skip over any broken links or incomplete job ads.

## The final complete code

<details>
<summary>Python code</summary>
{% highlight python %}  
{% raw %}
import time
import os
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from openpyxl import load_workbook

# Load Excel file containing LinkedIn job URLs
file_path = "/path/to/job_ads.xlsx"
df = pd.read_excel(file_path, skiprows=0, engine="openpyxl")

# Get job URLs from Excel sheet
job_links = df["Job URL"].tolist()

# Define a folder to save screenshots
screenshot_folder = "/path/to/save/screenshots"
os.makedirs(screenshot_folder, exist_ok=True)

## Using Chrome profile for LinkedIn login
# After see the login, do it manually for the first time using the code
# Chrome must not be running!

# Path to your Chrome user data directory (replace this with your path)
user_data_dir = "your Chrome user data directory/Google/Chrome"

# Path to the profile you want to use (this is usually in the 'Profile 1', 'Profile 2', etc. directories)
profile_dir = "Default"  # Change to the actual profile you're using

# Setup Chrome options
options = Options()
options.add_argument("--start-maximized")
options.add_argument(f"user-data-dir={user_data_dir}")
options.add_argument(f"profile-directory={profile_dir}")  # Load your LinkedIn login profile

# Just use webdriver.Chrome() without specifying chromedriver path
driver = webdriver.Chrome(options=options)

# Open LinkedIn (already logged in if using the default profile)
driver.get("https://www.linkedin.com") # Then login manually for the first time using the code

# Iterate through job URLs and scrape details
for index, job_url in enumerate(job_links):
    if not job_url:
        print(f"Skipping row {index + 1}: No valid LinkedIn job URL.")
        continue

    try:
        driver.get(job_url)
        time.sleep(3)  # Wait for the page to load

        # Click "See more" if available to expand job description
        try:
            # Wait until the "See more" button is clickable
            see_more_button = WebDriverWait(driver, 5).until(
                EC.element_to_be_clickable((By.XPATH, "//button[@aria-label='Click to see more description']"))
            )
            
            # Try normal click
            see_more_button.click()
            time.sleep(2)
            print("Clicked 'See more' successfully!")

        except Exception:
            try:
                # Try JavaScript click if normal click fails
                driver.execute_script("arguments[0].click();", see_more_button)
                print("Clicked 'See more' using JavaScript!")
                time.sleep(2)
            except Exception as e:
                print("No 'See more' button found or already expanded:", e)

        time.sleep(3)

        # Find all h2 elements for the "About the job" section
        h2_elements = driver.find_elements(By.TAG_NAME, "h2")

        # Locate the exact "About the job" section
        about_section = None
        for h2 in h2_elements:
            if "About the job" in h2.text:
                about_section = h2
                break

        # If found, scroll to it
        if about_section:
            driver.execute_script("arguments[0].scrollIntoView({block: 'start'});", about_section) # move to the top
            driver.execute_script("window.scrollBy(0, -130);")  # Move down a bit after scrolling
            print("Scrolled to 'About the job' section.")
            time.sleep(5)
        else:
            print("'About the job' section not found.")

        # Find the job description section
        job_desc = driver.find_element(By.CLASS_NAME, "job-view-layout")
        
        # Take a screenshot of the job details
        job_desc = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CLASS_NAME, "jobs-details")))
        job_desc.screenshot(screenshot_path)
        print(f"Saved screenshot for job ad {index + 1}: {screenshot_path}")

    except Exception as e:
        print(f"Row {index + 1}: Error opening job URL {job_url}. It may be removed. Error: {e}")
        continue

# Close the WebDriver
driver.quit()
{% endraw %}
{% endhighlight %} 
</details>

---

## The Difficulty of Scraping LinkedIn

While this project might seem straightforward, scraping LinkedIn comes with its own set of challenges. Here are some of the difficulties I encountered:

#### **1. LinkedIn's Anti-Scraping Measures**

LinkedIn, like many other websites, employs **anti-scraping techniques** to prevent bots from accessing its data. These measures can include:
- **CAPTCHAs**: Asking users to verify they are human.
- **Rate limiting**: Blocking or slowing down requests after a certain number of accesses.
- **Session timeouts**: Logging out users after periods of inactivity.

LinkedIn’s dynamic and personalized nature (e.g., custom pages based on your profile) also makes it challenging to scrape. Despite this, by using **Selenium** and automating a browser session, we can simulate human behavior (like clicking buttons and scrolling), making it harder for LinkedIn to detect that the activity is automated.

#### **2. Dynamic Content Loading**

LinkedIn job descriptions sometimes load dynamically, meaning they aren’t fully loaded when the page initially renders. This requires us to either:
- Wait for content to load using **WebDriverWait**.
- Scroll down the page to load the remaining content.

These steps add complexity to the script, as it needs to wait for elements to load before proceeding with further actions.

#### **3. Constantly Changing HTML Structure**

Web scraping relies on specific HTML structure identifiers (like class names, IDs, or XPath). LinkedIn’s HTML structure often changes, which means that scrapers can break when these identifiers are updated. For example, LinkedIn might change the name of a div that contains the job description. This requires continuous maintenance of the script to ensure it works correctly.

---

## Reflections on AI-Assisted Programming

While working on this project, I was fascinated by how much **ChatGPT** could help in writing and refining the code. However, one important takeaway is that **AI doesn't replace human thinking**—it augments it.

Here are a few of my personal reflections:

#### AI-Generated Code Isn't Perfect

The first version of the code generated by ChatGPT often worked as a **starting point**, but rarely as a complete, foolproof solution. It was up to me to:
- Describe the task **clearly and precisely**.
- Identify what parts of the code didn’t work as expected.
- Ask better follow-up questions.
- Iterate and refine.

In short, AI thrives when guided by a clear mind and a curious spirit.

#### Human Thinking Is Still Essential

AI can't "see" what’s happening on your browser screen or understand the nuance of LinkedIn's ever-changing HTML structure. It can’t click a button that doesn’t exist, nor can it know that an element loads five seconds later. It’s up to **you** to:
- Think critically.
- Test often.
- Debug carefully.
- Fine-tune the behavior until it works the way you need.

#### It’s a Process of Iteration

One of the biggest misconceptions about AI-assisted coding is that it gives you "the answer" instantly. The reality? It’s more like having a really fast-thinking assistant that **needs direction and feedback**. Most tasks—especially real-world tasks like web scraping—require **multiple iterations** before they’re stable and reliable.

#### It Demands Patience and Perfectionism

If you’re the kind of person who wants things to “just work,” it can be frustrating at first. But as I learned to patiently go through the debugging process, to tweak selectors, adjust scroll offsets, and wait for dynamic content to load, I realized this was all part of **becoming a better problem solver**.

It reminded me that **great code is crafted**, not copy-pasted.

## Final Word

In the age of AI-assisted programming, success doesn’t come from pressing a magic button. It comes from **pairing AI tools with human critical thinking**, curiosity, and a healthy dose of perfectionism.

For me, this project wasn’t just about saving LinkedIn job ads. It was about learning how to collaborate with AI to solve real problems—and how to stay engaged, creative, and persistent in the process.
