# DSAI3202 — Project Phase 1
## Toxicity Monitoring on Reddit Communities
#### Members: Lynn Younes 60107070 & Urooj Shah 60300832
---
### Introduction
People online post:  
- insults  
- threats  
- questionable opinions like "mint choco chip ice cream is good"

This creates unsafe environments and forces moderators to experience **rapid aging buffs**.

So we're gonna built a system to detect toxicity in real-time.

### Basically us, reading the dataset:

**Dataset:** "User123 said: *you absolute waste of oxygen—*"  
**Us:**  
![dog](./img/dawg.jpg)


This project builds an end-to-end cloud analytics pipeline that ingests Reddit community data, cleans and transforms it inside an Azure Lakehouse, engineers features, and visualizes toxicity trends using Power BI.

The goal is to measure community toxicity, sentiment behaviour, and interaction risks using modern text analytics techniques and scalable cloud infrastructure, while Reddit invents new slang every 3 business days.
*(A.K.A: We built a robot that reads comments so we don't have to. Because mental health.(Phase 2))*


The pipeline uses:
- Azure Data Lake Gen2 (Bronze → Silver → Gold)
- Azure Databricks (PySpark) for transformations
- Text feature engineering
- Power BI for dashboards and exploratory analysis

---

# Reddit API Access (InshaAllah Phase 2)
We use **PRAW**. (This will be used in Phase 2)

PRAW is like:

> "Reddit, can we please have… one (1) comment? Please? Gently?"

Reddit free tier lets us send **100 requests/min**, which works perfectly because:
- we're students  
- our budget is please 
- and we are NOT paying for API calls when we can barely pay for lunch 

---

### README Organization
This README is organized as follows:

1. [Project Directory Structure](#1-project-directory-structure)  
2. [Lakehouse Architecture](#2-lakehouse-architecture)  
   - I. [Bronze Layer](#i-bronze-layer)  
   - II. [Silver Layer](#ii-silver-layer)  
   - III. [Gold Layer](#iii-gold-layer)
3. [Final Dataset Schema](#v-final-dataset-schema)
4. [Power BI Dashboard](#3-power-bi-dashboard)   
5. [Challenges](#4-challenges)
6. [Conclusion](#4-conclusion)

---
### 1) Project Directory Structure
Below is the structure of this GitHub repository:
```
dsai3202-toxicity-monitor/
│
├── data/
│   ├── sample_cringeanarchy.json        # Sample data for demonstration
│   ├── sample_greatawakening.json
│   └── sample_mensrights.json
│
├── img/
│   ├── bronze_layer.png
│   ├── silver_layer.png
│   ├── gold_layer.png
│   ├── lakehouse_directory.png
│   └── Reddit EDA Dashboard.jpg
│
├── notebooks/
│   └── MADOC gold feature engineering.ipynb   # Main Databricks notebook where cleaning and feature engineering happens
│
├── src/
│
├── README.md                         # This file
└── requirements.txt                  # Python dependencies

```

### 2) Lakehouse Architecture
Our architecture follows the holy trinity of data engineering: **Bronze → Silver → Gold**,  
also known as:  
**"Raw Chaos → Slightly Less Chaos → Data Your Professor Might Actually Look At."**

#### I. Bronze Layer
The Bronze layer is where **raw, unfiltered, fully-feral data goes to chill**.

Source of data?

Remember that dataset paper website from the proposal, the MADOC project?  
They had datasets from:
- Reddit  
- Bluesky  
- and some other platform that felt like a social network from the multiverse  

So from their massive chaotic archive, we picked a few communities from reddit:
- **r/CringeAnarchy**  
- **r/greatawakening**  
- **r/MensRights**  
*(Fun fact: the fat people subreddit dataset got corrupted. Probably a sign from the universe.)*

Process:
- Created **Storage Account** `toxicitylake7032` (last 2 digits of both of our ID's), **Lakehouse**, and **raw/silver/gold** folders (just like the labs).
- Used **Azure Machine Learning** to run `wget` + `azcopy` to haul the data from the MADOC site to our Lakehouse.
- Dropped the untouched parquet files into **bronze/reddit/**.

Essentially, the Bronze layer is:

**Bronze layer be like:**  
![dumpster-fire](./img/dump.gif)


#### II. Silver Layer

The Silver layer is where the data stops being wild Reddit creature and starts becoming a citizen.

Since all MADOC subreddit files were already in **Parquet** (bless whoever made that decision),  
we didn't have to fight with CSVs, JSONs, or existential dread.

Using **Azure Data Factory**, we:
- created a pipeline  
- moved each subreddit parquet into **silver/reddit/**  
- merged them into one unified Reddit folder  


Silver layer = Bronze, but after drinking water and going to therapy.

#### III. Gold Layer

Welcome to the **Gold Layer**, where the dataset hits its personal villain arc.

In our Databricks notebook (yes, the one that slowly consumed our sanity), we:
- loaded the clean Silver data  
- added some simple but useful features like:
  - `content_length_words`
  - `toxicity_label`
  - `risk_level`
  - anything else that screams "this comment might be toxic"  
- didn't bother with SBERT embeddings because:
  - MADOC already comes with toxicity + sentiment labels  
  - and we're not trying to melt our Azure credits

We took the cleaned Silver data and engineered a set of features that make the dataset more useful for analysis and modeling. We filtered and standardized key columns, then created text-based features like total character count and word count to capture how long each piece of content is. We also generated a toxicity label by turning the toxicity score into a simple 0/1 indicator. On top of that, we built sentiment buckets (positive, neutral, negative) from the sentiment score, and created a risk-level feature that classifies each item as low, medium, or high risk based on toxicity, flags, and other strict-filtering rules. Altogether, we enriched the original Silver data and shaped it into a structured Gold dataset that’s ready for machine learning, reporting, or any downstream analysis.

Gold outputs stored under:  
**gold/reddit_features/*


### Final Dataset Schema  

Our final cleaned dataset includes:

| Column | Description |
|--------|-------------|
| `post_id` | Unique ID of the post |
| `user_id` | Who unleashed the content into the world |
| `parent_id` | ID of the parent post/comment in the thread |
| `parent_user_id` | The human (or bot?) behind the parent content |
| `platform` | Always Reddit |
| `community` | Source community |
| `interaction_type` | Was it a post? A comment? A reaction? Reddit has layers |
| `publish_ts` | Exact timestamp when the chaos went live |
| `publish_date_only` | Same timestamp, but date only |
| `content` | The raw, unfiltered Reddit text |
| `url` | Where this blessed content originally lived |
| `sentiment_vader` | VADER sentiment score (robot feelings detector #1) |
| `sentiment_textblob` | TextBlob sentiment score (robot feelings detector #2) |
| `subjectivity_textblob` | How opinionated the content is (according to TextBlob) |
| `toxicity_toxigen` | Toxicity score from the ToxiGen model |
| `strict_filter` | Whether this content trips the “uh oh” filter |
| `content_length_chars` | Hand-engineered length feature (characters) |
| `content_length_words` | Hand-engineered length feature (words) |
| `toxicity_label` | Final toxicity classification |
| `sentiment_bucket` | Sentiment grouped into buckets |
| `risk_level` | How ragebait-y the content is |


### Power BI Dashboard
For visualization:
- Connected Power BI Desktop directly to Azure (Data Lake storage). (this took a while) 
- Loaded the dataset from **gold/reddit_features/**.  
- Built dashboards showing:
  - toxicity levels  
  - sentiment overview  
  - subreddit comparisons  
  - comment behavior trends  

It's basically **Google Analytics for Drama™.**

Heres a screenshot of our dashboard. Wanted to add a link but kept crashing when we tried to publish. Lawd have mercy.

![dashboard](./img/reddit_eda_dashboard.png)

Power BI be like:  
"Your subreddit is 63% more toxic this week ✨. Maybe log off."

### Challenges
We encountered many bosses in this RPG adventure, including:

- Parquet files judging us silently  
- Azure ML refusing `wget` until we convinced it  
- Corrupted subreddit file (RIP to the fat people dataset 🕊️)  
- Data Factory pipelines running only when properly bribed  
- Spark notebooks that said "Im gonna stop you right there…"
- Connecting the Azure Blob Storage to Power BI took 47 business years
- Subreddits that emotionally damaged us during EDA  
- The moment we realized some comments should stay unread forever  

### Conclusion
In Phase 1, we successfully:

- built the full **Bronze → Silver → Gold** data pipeline  
- ingested real Reddit community datasets from MADOC  
- cleaned and engineered features in Databricks  
- generated a final usable dataset in **gold/reddit_features**  
- built a connected Power BI dashboard to visualize toxicity trends  

We laughed, we cried, we begged Azure to stop charging us.

Phase 2 will bring:
- Real-time ingestion (pray for us)  
- Live streaming dashboards (inshaAllah) 
- Possibly a built-in "bro seek therapy" classifier

![bravo](./img/clap.gif)
![bravo](./img/clap2.gif)
