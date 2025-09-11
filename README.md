# Puppeteer Worker 2

This is the **first worker (of three)** that scrapes URLs fed from a backend service (`chunkgeneratorforaimodel`) based on searched keywords.  
The workers run in parallel to share the scraping load, increasing the speed and efficiency of the process.

---

##  ^=^s  About This Package

- Functions as **2/3 of the scraper workload**.
- Consumes jobs (URLs + keywords) from **Kafka**.
- Uses **Puppeteer** to scrape the content of web pages.
- Filters and extracts text that contains the searched keywords.
- Sends results back to Kafka for further processing by the backend (`chunkgeneratorforaimodel`).
- Designed to run together with two other workers for parallel scraping.

---

##  ^z^y  ^o How It Works

1. **Kafka Consumer**  
   - Listens for messages on the `topuppeteerworker` topic.  
   - Each message contains ~1/3 of the total URLs to be scraped.

2. **Scraping Process** (`runScraper` in `puppeteerWorker.js`)  
   - Launches Puppeteer in headless mode.
   - Visits each URL.
   - Extracts page text.
   - Checks for presence of keywords from `topicsArray`.
   - If found, pushes a result in the format:

   {
 "text": "page content",
     "metadata": {
       "url": "https://example.com",
       "date": "2025-09-11T10:00:00.000Z",
       "sourcekb": "external",
       "searched": "your_query"
     }
   }

 ^=^r  Platform Requirements

At least 12 GB RAM

At least 4 CPU cores (Intel i5 or higher recommended)

Docker installed

 ^=^z^` How to Run

Download the Docker image

**docker pull ghcr.io/hendram/puppeteerworker2**


Start the container

**docker run -it -d --network=host ghcr.io/hendram/puppeteerworker2 bash**


Find your container name

**docker ps**
Example: pedantic_payne (your name will differ).

Enter the container

**docker exec -it pedantic_payne /bin/bash**


Run the service

**cd /home/browserconnworker2**
**node index.js**

##  ^=^t  Code Overview

- **`index.js`**  
   ^=^z^` Starts the worker, consumes jobs from Kafka, runs the scraper, and sends results back.

- **`kafka.js`**  
   ^=^t^l Handles Kafka connection, producer, and consumer setup.

- **`puppeteerWorker.js`**  
  - ^=^u   ^o Contains the `runScraper` function, which:
  -  ^=^v   ^o Launches Puppeteer (headless browser)
  -  ^=^l^p Visits URLs
  -  ^=^t^m Extracts and filters page content
  -  ^=^s  Returns results with metadata

---

##  ^|  Features & Functionality

 ^|^t  ^o Consumes jobs (**URLs + keywords**) from Kafka  
 ^|^t  ^o Uses **Puppeteer** to scrape web pages in headless mode  
 ^|^t  ^o Extracts page text and checks if it contains keywords  
 ^|^t  ^o Produces structured results with metadata (URL, date, source, searched keyword)  
 ^|^t  ^o Sends results back to Kafka for merging with outputs from other workers  
 ^|^t  ^o Designed for **distributed, scalable, and fast scraping**

---

##  ^=^s  Data Flow

    A[chunkgeneratorforaimodel] -->|Jobs| B[Kafka: "topuppeteerworker"]
    B --> C[Puppeteer Worker 2  ^=^u   ^o]
    C -->|Results| D[Kafka: "frompuppeteerworker"]
    D --> E[Backend merges with Worker 2 & 3]

