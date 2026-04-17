PRODUCT REVIEW ANALYSIS PIPELINE - README

60.002 A.I. Applications and Design | SUTD

Members:

Styliani Ionna Tzertzeveli  (1011224)

Suheera Banu                (1009023)

Shannon Hi                  (1009048)

Stephanie Ho Yen Yuin       (1009356)

Vadora Tang                 (1009309)



**PROJECT OVERVIEW:**

This project implements an end-to-end AI-driven pipeline that automatically collects product reviews from the web, performs sentiment analysis on specific product features, and generates structured design requirements. It was developed as a consulting tool to identify product design opportunities from real consumer feedback.



The pipeline was demonstrated using "Toilet Bowls" as the target product, but is designed to be fully product-agnostic - it can be redeployed for any product category by simply changing the input product name.



**FILES INCLUDED:**

1\. DAI\_product\_review\_analysis.ipynb

&#x20;  - Main Colab notebook containing the full pipeline code.

&#x20;  - Run this file end-to-end in Google Colab.



2\. output/Toilet\_Bowls/

&#x20;  - 01\_raw\_reviews.xlsx

&#x20;    Raw review data collected from the web via SerpAPI. Contains fields:

&#x20;    source, rating, title, content, date.



&#x20;  - product\_analysis.xlsx

&#x20;    Master output file with 4 sheets:

&#x20;      • Raw Reviews       - unprocessed review data

&#x20;      • Feature Sentiment - per-feature sentiment scores and star ratings

&#x20;      • Product Requirements - GPT-4o generated design requirements

&#x20;      • Summary           - high-level run statistics



&#x20;  - feature\_sentiment\_chart.png

&#x20;    Horizontal bar chart visualising the 1–5 star sentiment rating for

&#x20;    each product feature.



3\. keys/ (won't be in file since secret key)

&#x20;  - open\_ai2.txt     - OpenAI API key

&#x20;  - serpapi\_key.txt  - SerpAPI key



**HOW TO RUN:**

Prerequisites:

&#x20; - Google Colab account with Google Drive mounted

&#x20; - Valid API keys for OpenAI and SerpAPI stored in the keys/ folder



Step 1: Open DAI\_product\_review\_analysis.ipynb in Google Colab.



Step 2: Mount Google Drive and set the working directory to (won't work with Shared With Me so please clone it):

&#x20;       /content/drive/MyDrive/AIDS\_proj2/



Step 3: Ensure API key files are placed at:

&#x20;       keys/open\_ai2.txt

&#x20;       keys/serpapi\_key.txt



Step 4: Run all cells in order. When prompted, enter a product name

&#x20;       (e.g. "Toilet Bowls").



Step 5: Results are saved automatically to:

&#x20;       output/<product\_name>/product\_analysis.xlsx



Dependencies:

transformers, torch, sentencepiece, openpyxl, serpapi, google-search-results, requests, pandas, matplotlib, numpy



**PIPELINE STAGES:**

Stage 1 - User Input

&#x20; The user enters a product name. The pipeline uses this to scope alldownstream data collection and analysis.



Stage 2 - Data Collection

&#x20; SerpAPI queries Google Search using 6 varied search queries (e.g. "{product} review", "{product} user review pros cons") and collects up to 50 organic result snippets as proxy reviews.

&#x20; Raw data is saved to 01\_raw\_reviews.xlsx.



Stage 3 - Feature Identification

&#x20; GPT-4o is prompted to generate 8–10 key product features that users typically discuss in reviews (e.g. flush efficiency, ease of cleaning).

&#x20; This eliminates manual/subjective feature selection. You can still manually enter features by putting them into the features = \[] array and setting USE\_AUTO\_FEATURES = False;



Stage 4 - Sentiment Analysis

&#x20; The HuggingFace model cardiffnlp/twitter-roberta-base-sentiment (RoBERTa-based) is used to analyse sentiment at the sentence level.

&#x20; Each review is split into sentences; sentences are matched to features by keyword; matched sentences are scored as:

&#x20;   LABEL\_0 (Negative) → -1

&#x20;   LABEL\_1 (Neutral)  →  0

&#x20;   LABEL\_2 (Positive) → +1

&#x20; Scores are weighted by model confidence.



Stage 5 - Feature Rating

&#x20; Per-feature sentence scores are averaged and mapped to a 1–5 star

&#x20; rating using the formula:

&#x20;   Star Rating = ((avg\_score + 1) / 2) × 4 + 1

&#x20; Positive and negative mention percentages are also calculated.



Stage 6 — Requirements Generation

&#x20; GPT-4o is given the sentiment summary and up to 30 raw reviews and prompted to produce 10–15 structured, measurable product requirements in the format: \[Category] Requirement description.



**Feature results summary (Toilet Bowls):**

&#x20; comfort height      4.2 ★  (7 mentions)

&#x20; flush efficiency    4.1 ★  (18 mentions)

&#x20; design aesthetics   4.1 ★  (5 mentions)

&#x20; price               3.9 ★  (2 mentions)

&#x20; ease of cleaning    3.8 ★  (39 mentions)

&#x20; bowl shape          3.8 ★  (26 mentions)

&#x20; water usage         3.7 ★  (9 mentions)

&#x20; noise level         3.0 ★  (4 mentions)

&#x20; installation process 3.0 ★ (2 mentions)

&#x20; durability          3.0 ★  (0 mentions)



**LIMITATIONS:**

\- Sample size of 47 reviews may not represent global user opinions.

\- Review snippets from search results are shorter than full product reviews which may reduce sentiment signal quality.

\- Keyword matching splits multi-word features into individual words, which may produce false matches (e.g. "ease of cleaning" matches any sentence

&#x20; containing "ease", "of", or "cleaning" independently).

\- Features with fewer than 5 mentions (e.g. installation process, durability) have statistically unreliable sentiment scores.

\- The RoBERTa model was pre-trained on tweets so it might not fully capture nuanced language in longer, more formal review text.

\- GPT-4o generated requirements may contain hallucinated specifics (e.g. price ranges) not directly supported by source reviews.

