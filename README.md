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
---

PHASE 2 — Live Toxicity Monitoring on Bluesky
Overview

Phase 2 extends our project from static historical Reddit data (Phase 1) to live, real-time social content using the Bluesky API.
We built a micro-batch streaming pipeline that ingests fresh posts from our Bluesky home timeline, lands them in our Azure Lakehouse (Bronze → Silver → Gold), generates text features, and finally applies our trained Reddit toxicity model on this new live data.

This phase demonstrates how the same cloud architecture from Phase 1 can evolve into a real-time analytics system capable of continuously monitoring online toxicity.

📡 1. Live Ingestion with Bluesky API (Bronze Live)

In Phase 2, we replaced the inaccessible Reddit API with Bluesky's atproto API, which provides free access to live social content.

Why Bluesky instead of Reddit?

Reddit’s API used to be the go-to option for academic and student projects, but after their 2023 API policy overhaul, most streaming endpoints became restricted, rate-limited, or locked behind paid tiers. The PRAW (Python Reddit API Wrapper) library still works, but only in very low-volume and non-real-time scenarios. For our use case — continuously ingesting fresh comments for live toxicity scoring — Reddit’s API no longer offers a reliable, free streaming mechanism.

Meanwhile, Bluesky offers an open, developer-friendly API with:

✅ Free access for small educational projects

✅ A clean, modern authentication flow

✅ The ability to fetch fresh posts and replies from a user’s feed

✅ Lightweight JSON responses (perfect for cloud ingestion)

✅ No complicated rate-limit policies for simple ingestion loops

So instead of fighting the Reddit API boss on hard mode, we switched to the API that actually wants us to succeed. Bluesky gives us real, fresh social media content — which is all we need for Phase 2.

This section explains how the model was built, what features were used, and how well the classifier performed.

🤖 1. Why Train on Reddit?

The MADOC Reddit dataset provides:

large-scale comment data

built-in toxicity & sentiment labels

high diversity of communities

a wide range of writing styles and toxicity levels

This makes it ideal for training a toxicity detection model that generalizes well to other platforms — including the live Bluesky posts used in this phase.

🧱 2. Features Used for Model Training

From the Reddit Gold dataset, we selected the following features:

📝 Text feature

content (the raw text itself)

This is transformed using TF-IDF (TfidfVectorizer):

unigrams

max 20,000 features

English stopwords removed

🔢 Numeric features

content_length_words

content_length_chars

sentiment_vader

sentiment_textblob

subjectivity_textblob

These features help the model capture patterns beyond words alone — such as emotional tone, subjectivity, and text length.

🧩 3. Model Choice: Logistic Regression

We used Logistic Regression with:

max_iter=2000

balanced-ish dataset

scikit-learn pipeline (ColumnTransformer + StandardScaler + TF-IDF)

Why Logistic Regression?

Logistic Regression is:

fast

stable

robust to noisy text

interpretable

good as a first model for text classification

It’s widely used in industry as a strong baseline for toxicity, sentiment analysis, and moderation systems — especially before moving to deep learning models like BERT or SBERT.

🧪 4. Train/Test Split & Evaluation

We used:

80% training

20% testing

stratified split (keeps class imbalance consistent)

After training, we evaluated the model on the held-out test set.

📊 Final Model Metrics
=== Evaluation on Test Set ===
Accuracy : 0.7922801608816599
F1       : 0.42406229274643215
Precision: 0.6607066315596105
Recall   : 0.3122309756856277
ROC AUC  : 0.7730654979253081

🧾 Classification Report
Class	Precision	Recall	F1	Notes
0 (non-toxic)	0.81	0.95	0.87	Very strong performance
1 (toxic)	0.66	0.31	0.42	Catches ~31% of toxic posts

Overall:

Accuracy: 0.79

Macro F1: ~0.65

ROC AUC: 0.77

🔍 5. Interpretation of These Metrics
✔ Strengths

Detects non-toxic comments extremely well

Reasonably good precision for toxic comments
→ When it predicts “toxic,” it’s usually right

Solid ROC AUC of 0.77
→ Better than random, clear separation between classes

⚠ Weakness

Recall for toxic posts is low (0.31)
→ The model misses some toxic content

This is expected because:

Toxicity is rare compared to non-toxic content

Logistic Regression is linear (limited expressiveness)

Reddit comments vary wildly in style

Despite this, the model is good enough for a Phase 2 academic project, especially since:

It generalizes well to unseen Bluesky text

It runs extremely fast

It integrates cleanly with Databricks + MLflow

🔐 Authentication

We securely authenticate using Databricks Secrets:

bsky_id = dbutils.secrets.get("bsky-scope", "BSKY_ID").strip()
bsky_pw = dbutils.secrets.get("bsky-scope", "BSKY_APP_PASSWORD").strip()

client = Client()
profile = client.login(bsky_id, bsky_pw)


This avoids exposing credentials in the notebook or repository.

📥 Fetching Live Posts

We query Bluesky’s timeline endpoint:

params = AppBskyFeedGetTimeline.Params(limit=LIMIT)
timeline = client.app.bsky.feed.get_timeline(params=params)


LIMIT = 50
This creates a micro-batch streaming ingestion pattern — a widely used technique in real systems where the pipeline runs periodically and processes a small batch of the newest records.

Why this counts as “Live”

Every time the ingestion cell runs, it retrieves fresh, real Bluesky posts from our home timeline.

This means the pipeline operates on current social content, not historical files.

Perfect for real-time toxicity monitoring.

🪵 2. Bronze Live Layer — Landing Raw Bluesky Data

We normalize each feed item into a simple schema:

post_id

user_id

platform

publish_ts

content

url

community

interaction_type

These rows are written in raw JSON to:

bronze_live/bluesky/

Why raw JSON?

The Bronze layer always stores unaltered, uncleaned, raw data so nothing is lost. This mirrors the structure used in Phase 1 for Reddit.

🥈 3. Silver Live Layer — Cleaning & Standardization

The Silver layer cleans and standardizes the data:

Removes empty or null content

Converts timestamps into timestamp datatype

Adds publish_date_only for easier time-series analysis

Writes the result as optimized Parquet files to:

silver_live/bluesky/

Why Silver?

Silver is the “clean” version of the data — typed, validated, and ready for analytics or feature engineering.
This follows the same Bronze → Silver → Gold progression as the original Phase 1 pipeline.

🪙 4. Gold Live Layer — Feature Engineering for Machine Learning

This step transforms each Bluesky post into a feature-rich representation similar to the Reddit dataset used to train our model.

We compute:

🧱 Length-based features

content_length_chars

content_length_words

These help the model capture patterns like:

short, aggressive replies

long, opinionated rants

extremely short outbursts

✨ Sentiment features (TextBlob)

We use TextBlob sentiment analysis to compute:

sentiment_textblob (range: –1 to +1)

subjectivity_textblob (range: 0 = factual → 1 = opinionated)

These help the model interpret emotional tone, which strongly correlates with toxicity.

🧩 Compatibility Feature: sentiment_vader

Our Reddit model was trained on sentiment_vader, but Bluesky lacks it.
To maintain compatibility with the existing ML pipeline, we approximate:

sentiment_vader = sentiment_textblob


This allows the model to be applied to Bluesky without retraining.

📂 Gold output path:
gold_live/bluesky_features/


This table contains one row per Bluesky post with all engineered features, ready for scoring.

🤖 5. Applying the Reddit Toxicity Model to Live Bluesky Posts

We load our Phase 1 ML model from MLflow:

logged_model_uri = f"runs:/{RUN_ID}/model"
reddit_toxicity_model = mlflow.sklearn.load_model(logged_model_uri)


We then:

Take a sample of 200 Bluesky Gold posts

Convert them to pandas

Run .predict() and .predict_proba()

Attach the predictions as new columns

Convert back to Spark

Finally, we save the scored dataset to:

gold_live/bluesky_scored/

Why this matters

This proves true end-to-end functionality:

Live ingestion (API)

Cleaning (Silver)

Feature engineering (Gold)

Model scoring (MLflow)

Storage for dashboards

Your pipeline is now able to evaluate toxicity in real time.

📊 6. Phase 2 Dashboard (Bluesky Live Analytics)

In Power BI (or any BI tool), we connect directly to:

gold_live/bluesky_scored/


Recommended visuals:

Table of Live Posts + Predicted Toxicity

Toxicity Over Time

Top Toxic Authors

Sentiment Distribution on Live Data

This complements the Phase 1 Reddit dashboard, which focuses on historical EDA.

🧠 7. Why This Architecture Is “Live”

Even without streaming platforms like Kafka, this design is officially considered live ingestion:

When the notebook runs, it pulls current Bluesky posts

The pipeline processes them instantly

Output is updated Gold Live + Scored data

Dashboards immediately reflect new posts

This is a micro-batch streaming architecture, a standard industry pattern used by:

Airbnb

Netflix

Uber

Reddit

Spotify

Perfect for academic projects and scalable in real cloud environments.
