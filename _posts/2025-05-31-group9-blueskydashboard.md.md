---
layout: post
title: "Real-Time Social Media Analytics on Bluesky"
author: Fabian Spühler & Jonne van Dijk
---

Access to social media data is increasingly restricted. Platforms like X (formerly Twitter), Facebook, or TikTok limit or monetize API access, making real-time research difficult or impossible.

[Bluesky](https://blueskyweb.xyz/) breaks that pattern. Thanks to its decentralized architecture ([AT Protocol](https://atproto.com/)) and the [Jetstream API](https://github.com/bluesky-social/atproto/tree/main/packages/api), it offers open access to a continuous stream of public posts — no paywalls, no API keys. We used this opportunity to build a real-time analytics pipeline from scratch, using only open-source tools.

## Our Pipeline at a Glance

We designed a modular, containerized system that ingests, enriches, stores, and visualizes millions of Bluesky posts per day. The pipeline includes:

- [**Jetstream API**](https://github.com/bluesky-social/atproto/tree/main/packages/api) + [**Apache Kafka**](https://kafka.apache.org/) for ingestion and event routing  
- [**langdetect**](https://pypi.org/project/langdetect/) for language detection (CPU)  
- [**distilbert-base-uncased-finetuned-sst-2-english**](https://huggingface.co/distilbert-base-uncased-finetuned-sst-2-english) for sentiment analysis (GPU)  
- [**yangheng/deberta-v3-base-absa-v1.1**](https://huggingface.co/yangheng/deberta-v3-base-absa-v1.1) for ABSA (GPU)  
- [**Apache Parquet**](https://parquet.apache.org/) for long-term raw data storage  
- [**TimescaleDB**](https://www.timescale.com/) for structured, time-series-optimized metadata  
- [**Streamlit**](https://streamlit.io/) for live ABSA sentiment visualization

![Architecture](./assets/img/2025-05-31-group9-architecture.png)

All components are orchestrated via [Docker Compose](https://docs.docker.com/compose/) and can scale horizontally. Under load, the system processed up to **12,000 posts per minute**.

## Dashboard and ABSA Focus

Our dashboard, available at [bd.fs0.ch](https://bd.fs0.ch), focuses exclusively on **aspect-based sentiment analysis**. It highlights how entities (like companies, products, or people) are discussed over time — using fine-grained, entity-level sentiment scores.

This is especially useful for tracing public opinion around specific topics without relying on hashtags or keyword search.

## Handling Large Threads

Thanks to our thread reconstruction logic (based on `parent_id`, `root_id`, and `repost_id`), we can rebuild and analyze entire conversation trees. In one case, we extracted and processed a thread of **over 10,000 posts** discussing large language models (LLMs), including nested replies and reposts.

The result: a complete picture of how sentiment evolved, who participated, and when the discussion peaked.

## What’s Next?

We’re preparing two major extensions:

- Storing post relationships and semantic representations in a **graph or vector-based system**, optimized for time-sensitive queries  
- Hosting our own **Jetstream endpoint**, or connecting directly to the **AT Protocol**, to gain deeper control over ingestion and streaming behavior

Both directions aim to expand the analytical scope and maintain full autonomy over the data pipeline.

## Try It Yourself

- 🔎 **Live dashboard**: [bd.fs0.ch](https://bd.fs0.ch)  
- 💻 **Codebase (cleaned)**: [github.com/BDP25/BlueskyDashboard](https://github.com/BDP25/BlueskyDashboard)

We built this system to show that open, real-time social media analytics are still possible — and maybe more important than ever.

