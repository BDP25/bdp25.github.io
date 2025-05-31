---
layout: post
title: "Fraud Detection - Between Textbook and Reality"
author: Noa Flückiger, Fabio Kost, David Schiess
---

# Fraud Detection - Between Textbook and Reality

## Machine Learning Project Lifecycle

At university, you learn how a Big Data project should ideally run. You follow the machine learning lifecycle: start by defining the problem, process the data, build clean features, train models, and if all goes well, you end up with a functioning algorithm you can proudly solve your problem with.

Naturally, the spotlight is on modeling; after all, that’s what you’re studying Data Science for. Models are exciting, powerful, mathematically elegant.

That’s the theory.

In practice, things look very different. Especially when you’re not just doing a university project, but working with an industry partner and suddenly realize: the everyday reality of a machine learning project is 80% data chaos, hardware limits, shifting requirements, and constant adjustments. Welcome to the real world.

## What We (Can’t) Talk About

Ultimately, we’re not allowed to share all the details of our project due to privacy agreements; another real-life factor in applied data products. But here’s what we can say: we worked with millions of billing records and were tasked with uncovering suspicious patterns that show potential signs of fraudulent behavior among billers.

Our first idea was a classic clustering algorithm. The concept was simple and elegant: doctors with similar billing behavior would naturally group together, and anything that didn’t fit would stand out as an outlier.

But as we dug deeper into the research, we found a more exciting direction: **sequence models**, specifically **transformer architectures**, that could analyze billing histories as temporal sequences. We found that some people already tried this technique for different fraud detection systems, but it isn’t yet an over-explored approach. The more we read, the more it became clear: this is what we wanted to try. Not just because they seemed more promising, but also because it’s pretty exciting to build your own transformers.

So far, until the brainstorming phase, everything worked out well — but it was also where our nice, clean lifecycle dream came to a screeching halt.

## When Everything Keeps Changing

What we originally saw as fixed parameters — data structure, research question, project goals — turned out to be anything but stable. Our partner kept changing how we could access and what data we could access, which impacted how we could build our model.

Our well-planned machine learning lifecycle started to feel more like a machine learning **merry-go-round**: the data we had just spent days cleaning suddenly came in a different format; features we had carefully engineered no longer matched the revised objectives.

We planned in days for data preparation, but ended up needing weeks.

Looking back, we didn’t adapt the project to fit the lifecycle; **we adapted the lifecycle to fit the project.**

After several frustrating “back to square one” moments, we finally accessed and prepared our dataset. We built a data pipeline that sorted treatment histories per biller chronologically and encoded them as numerical sequences. Using these, we trained **transformer-based encoder-decoder** to detect common patterns — and flag anything significantly different with a high anomaly score.

This gave us a way to perform statistical analyses based on sequence-modeled data — helping to uncover suspicious billing behavior.

## The eventual pipeline
```mermaid
---
config:
  layout: dagre
---
flowchart TB
 subgraph subGraph0["Data Layer"]
        DB[("SQL Database<br>Multi-GB Medical Records")]
        PQ["Parquet Files<br>Columnar Format"]
  end
 subgraph subGraph1["Feature Extraction Pipeline (feature_extractor.py)"]
        DS["DoctorDataset<br>IterableDataset"]
        FE["DoctorFeatureExtractor<br>Chunked Processing"]
        CACHE[("Feature Cache<br>doctor_features.pkl")]
  end
 subgraph subGraph2["Feature Engineering"]
        AGG["Doctor Aggregation<br>• Billing patterns<br>• Treatment frequencies<br>• Sequence extraction"]
        META["Metadata Processing<br>• Invoice types<br>• Tariff information<br>• Patient context"]
        SEQ["Sequence Generation<br>• Chronological treatments<br>• Patient-level sequences<br>• Specialty grouping"]
  end
 subgraph subGraph3["ML Training Pipeline (transformer_model.py)"]
        LOAD["Feature Loading"]
        SPEC["Specialty Grouping<br>Cardiology, Dermatology, etc."]
        TOK["Tokenization<br>Treatment codes → indices"]
        DS2["TreatmentSequenceDataset<br>Metadata integration"]
  end
 subgraph subGraph4["Model Architecture"]
        TRANS["CTXTransformer<br>Autoencoder"]
        EMB["Embedding Layer<br>+ Positional Encoding"]
        MLP["Metadata MLP<br>Categorical + Continuous"]
        ATTN["Multi-Head Attention<br>Sequence Modeling"]
        DEC["Decoder<br>Reconstruction Loss"]
  end
 subgraph subGraph5["Training & Evaluation"]
        TRAIN["Per-Specialty Training<br>Adam Optimizer<br>CrossEntropy Loss"]
        EVAL["Reconstruction Error<br>Per-sequence scoring"]
        AGG2["Doctor-level Aggregation<br>Mean anomaly scores"]
  end
 subgraph subGraph6["Output & Results"]
        JSON["JSON Results<br>transformer_scores_*.json"]
        RANK["Anomaly Ranking<br>Top diverging doctors"]
  end
 subgraph subGraph8["Configuration & Control"]
        CONFIG["Configuration<br>• Column mappings<br>• Chunk sizes<br>• Test/Full modes"]
  end
    DB --> PQ
    PQ --> DS
    DS --> FE
    FE --> CACHE & AGG & META & SEQ
    CACHE --> LOAD
    LOAD --> SPEC
    SPEC --> TOK
    TOK --> DS2
    DS2 --> TRANS
    TRANS --> EMB & MLP & TRAIN
    EMB --> ATTN
    MLP --> ATTN
    ATTN --> DEC
    TRAIN --> EVAL
    EVAL --> AGG2
    AGG2 --> JSON
    JSON --> RANK
    CONFIG -.-> DS & FE & LOAD
     DB:::datastore
     PQ:::datastore
     DS:::processing
     FE:::processing
     CACHE:::datastore
     AGG:::processing
     META:::processing
     SEQ:::processing
     LOAD:::processing
     SPEC:::processing
     TOK:::processing
     DS2:::processing
     TRANS:::ml
     EMB:::ml
     MLP:::ml
     ATTN:::ml
     DEC:::ml
     TRAIN:::ml
     EVAL:::ml
     AGG2:::ml
     JSON:::output
     RANK:::output
     CONFIG:::config
    classDef datastore fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef processing fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef ml fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef output fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef config fill:#fce4ec,stroke:#880e4f,stroke-width:2px
```

## Comparing to Real Life

Of course, we had to know: **does it work?** We compared our results with cases that had already been flagged manually or identified through business rules by our partner. As expected many of our anomalies matched existing suspicions. We also uncovered a few new candidates.

It became clear just how valuable regular exchange with the partner was. Only their internal knowledge like specific exceptions or business context allowed us to interpret our model results in a meaningful way.

## The Model Was Cool - but Only One Part of the Job

The Transformer model was definitely a highlight. It’s impressive how such models can detect patterns, especially in an area where natural language, numbers, and behavioral patterns are so widespread and intertwined.

But as exciting as modeling is - it only made up about **30%** of our project time. The rest? **Data wrangling. Troubleshooting. Restructuring.** And then some more data wrangling.

We were looking forward to building a model. What we got was a crash course in data engineering, project flexibility, and communication.

## What We Learned

Our most important take-aways are:

- **Flexibility beats planning.** No plan survives first contact with real data.  
- **Time estimates are just that - estimates.** Feature engineering always takes longer than you think.  
- **First ideas aren’t always the best.** We’re glad we moved past our initial clustering approach.  
- **Interpretation is hard** Data doesn’t lie - but it won’t tell the truth unless you ask the right questions.  

In the end, our project may not have been “textbook perfect.” But it was **real**. And that’s exactly what made it so valuable, especially to our industry partner.