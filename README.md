# Product Review Analysis Pipeline
> 60.002 A.I. Applications and Design | SUTD

An end-to-end AI-driven pipeline that automatically collects product reviews from the web, performs sentiment analysis on specific product features, and generates structured design requirements - built as a consulting tool to identify product design opportunities from real consumer feedback.

The pipeline was demonstrated using **Toilet Bowls** as the target product. (This can be changed when asked for input)

---

## Team Members

| Name | Student ID |
|---|---|
| Styliani Ionna Tzertzeveli | 1011224 |
| Suheera Banu | 1009023 |
| Shannon Hi | 1009048 |
| Stephanie Ho Yen Yuin | 1009356 |
| Vadora Tang | 1009309 |

---

## Pipeline Overview

```
User Input → Data Collection → Feature Identification → Sentiment Analysis → Feature Rating → Requirements Generation
```


| Stage | Description |
|---|---|
| 1. User Input | Enter a product name to scope all downstream analysis |
| 2. Data Collection | SerpAPI queries Google Search across 6 varied queries, collecting up to 50 review snippets |
| 3. Feature Identification | GPT-4o auto-generates 8–10 key product features users typically discuss |
| 4. Sentiment Analysis | RoBERTa (`cardiffnlp/twitter-roberta-base-sentiment`) scores sentences per feature |
| 5. Feature Rating | Per-feature scores averaged and mapped to a 1–5 star rating |
| 6. Requirements Generation | GPT-4o produces 10–15 structured, measurable product design requirements |

---

### Detailed Pipeline Stages

**Stage 1 - User Input**  
The user enters a product name. The pipeline uses this to scope all downstream data collection and analysis.

**Stage 2 - Data Collection**  
SerpAPI queries Google Search using 6 varied search queries (e.g. `"{product} review"`, `"{product} user review pros cons"`) and collects up to 50 organic result snippets as proxy reviews.  
Raw data is saved to `01_raw_reviews.xlsx`.

**Stage 3 - Feature Identification**  
GPT-4o is prompted to generate 8–10 key product features that users typically discuss in reviews (e.g. flush efficiency, ease of cleaning).  
This eliminates manual/subjective feature selection. You can still manually enter features by putting them into the `features = []` array and setting `USE_AUTO_FEATURES = False`.

**Stage 4 - Sentiment Analysis**  
The HuggingFace model `cardiffnlp/twitter-roberta-base-sentiment` (RoBERTa-based) is used to analyse sentiment at the sentence level.  
Each review is split into sentences; sentences are matched to features by keyword; matched sentences are scored as:
- `LABEL_0` (Negative) → **-1**
- `LABEL_1` (Neutral) → **0**
- `LABEL_2` (Positive) → **+1**

Scores are weighted by model confidence.

**Stage 5 - Feature Rating**  
Per-feature sentence scores are averaged and mapped to a 1–5 star rating using the formula:

> **Star Rating = ((avg_score + 1) / 2) × 4 + 1**

Positive and negative mention percentages are also calculated.

**Stage 6 - Requirements Generation**  
GPT-4o is given the sentiment summary and up to 30 raw reviews and prompted to produce 10–15 structured, measurable product requirements in the format:  
`[Category] Requirement description.`

---

## Repository Structure

```
.
├── DAI_product_review_analysis.ipynb   # Main Colab notebook
├── output/
│   └── Toilet_Bowls/
│       ├── 01_raw_reviews.xlsx         #raw review data (source, title, content, date)
│       ├── product_analysis.xlsx       #master output (4 sheet tabs: Raw Reviews, Feature Sentiment, Requirements, Summary)
│       └── feature_sentiment_chart.png #bar chart
└── keys/                               #API keys (will not be included in this repo)
    ├── open_ai2.txt
    └── serpapi_key.txt
```

---

## Setup & Usage

### Prerequisites
- Google Colab account with Google Drive access
- Valid API keys for [OpenAI](https://platform.openai.com/) and [SerpAPI](https://serpapi.com/)

### Step 1 - Place API keys in Google Drive

Create a `keys/` folder in your Drive at the path below and add your key files:
```
/content/drive/MyDrive/AIDS_proj2/keys/open_ai2.txt
/content/drive/MyDrive/AIDS_proj2/keys/serpapi_key.txt
```
> ⚠️ `keys/` folder not committed to GitHub. It is listed in `.gitignore` just in case.

### Step 2 - Open the notebook

Open `DAI_product_review_analysis.ipynb` in Google Colab.

> Note: Mount from **My Drive**, not "Shared With Me" - clone the repo to your own Drive first.

### Step 3 - Set working directory

The notebook will `cd` into:
```
/content/drive/MyDrive/AIDS_proj2/
```

### Step 4 - Run all cells

When prompted, enter a product name (e.g. `Toilet Bowls`). Results are saved automatically to:
```
output/<product_name>/product_analysis.xlsx
```

### Dependencies

```bash
pip install transformers torch sentencepiece openpyxl serpapi google-search-results requests pandas matplotlib numpy
```

---

## Feature Results - Toilet Bowls

| Feature | Rating | Mentions |
|---|---|---|
| comfort height | ★ 4.2 | 7 |
| flush efficiency | ★ 4.1 | 18 |
| design aesthetics | ★ 4.1 | 5 |
| price | ★ 3.9 | 2 |
| ease of cleaning | ★ 3.8 | 39 |
| bowl shape | ★ 3.8 | 26 |
| water usage | ★ 3.7 | 9 |
| noise level | ★ 3.0 | 4 |
| installation process | ★ 3.0 | 2 |
| durability | ★ 3.0 | 0 |

---

## Limitations

- Sample size of 47 reviews may not represent global user opinions
- Search result snippets are shorter than full reviews, which may reduce sentiment signal quality
- Keyword matching splits multi-word features into individual words, potentially causing false matches (e.g. "ease of cleaning" matches any sentence containing "ease", "of", or "cleaning" independently)
- Features with fewer than 5 mentions have statistically unreliable sentiment scores
- GPT-4o generated requirements may contain hallucinated specifics (e.g. price ranges) not directly supported by source reviews
