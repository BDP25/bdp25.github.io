---
layout: post
title: "insurXtract: AI-Powered Insurance Policy Extraction Platform"
author: Michael Güntensperger, Philipp Schiess, Selina Schlegel
---

# Automating the Pain: How AI Transforms Insurance Document Processing in Switzerland

**Manual insurance paperwork is stuck in the last century. Can AI finally fix it without compromising data privacy?**

---

**Picture this**: You're switching your car insurance. You send in your old policy PDF. Behind the scenes, an overworked consultant opens the document, scans for key details like premiums, deductibles, and coverage limits, and retypes them into the system. Multiply that by thousands of customers every month and you get a good sense of the inefficiency at play.

But this isn’t only a productivity issue. It’s costly, error-prone, and clearly outdated — especially as customers increasingly expect fast, digital services. It’s exactly the kind of challenge that **AI is built to solve**.

---

## From PDFs to Policy Insights, Automatically

As part of the Big Data module in the BSc Data Science program at ZHAW, we developed InsuXtract: a scalable pipeline that automatically extracts structured data from unstructured insurance documents.

Our goal was to tackle a very real, very local problem. Swiss insurers are overwhelmed by paperwork, but strict data privacy laws mean most international AI tools aren’t an option.

InsuXtract processes both native and scanned PDFs, extracting key information like contract terms, coverage amounts, and the name of the insurer. This data is then made available to consultants through a web-based dashboard, where it can be reviewed, compared, and managed with ease.

The real challenge, though, wasn’t just getting the system to read documents. It had to do so accurately, intelligently, and in full compliance with Swiss data protection laws — all while scaling efficiently and running multiple microservices without failure.

<p align="center">
<img src="/assets/img/2025-05-30-group12-insurXract.png" width="700" alt="Core Concept">
</p>

---

## What Powers the System?

Here’s a quick breakdown of how InsuXtract works:

**Step 1: Upload**  
Customers receive a secure, tokenized link to upload their documents. No login is required.

**Step 2: Text extraction**  
Digital PDFs are parsed using [PyPDF2](https://pypi.org/project/PyPDF2/), while scanned ones go through [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) at 600 DPI, optimized for German.

**Step 3: NLP classification**  
A lightweight NLP model (Flair) determines whether the document is a policy or general terms (AVB) and identifies key details like the insurer and customer name.

**Step 4: LLM-based extraction**  
With tailored prompts for Swiss insurers like Mobiliar and Helvetia, [Meta’s LLaMA 3](https://ai.meta.com/llama/) extracts structured fields in JSON format. These are then mapped to a shared schema.

**Step 5: Sync and review**  
The extracted data is saved in a PostgreSQL database and displayed via a React-based consultant dashboard.

---

## How Well Does It Scale?

Performance testing showed that the system could handle up to **4.5 documents per minute** when running three LLM containers in parallel — almost three times faster than the single-threaded baseline.

Accuracy was also strong, especially with native PDFs:

| PDF Type | LLM Extraction Accuracy |
|----------|-------------------------|
| Native   | 93.8%                   |
| Scanned  | 82.8%                   |

Boolean fields (such as "covered: yes/no") were most affected by OCR issues. Nonetheless, numeric and textual fields held up well, even in lower-quality scans.

---

## Privacy by Design, for Switzerland

Most intelligent document platforms rely on cloud services hosted abroad, which conflicts with the Swiss Federal Act on Data Protection (DSG).

InsuXtract was designed from the ground up to ensure full data residency within Switzerland. All processing occurs locally or in certified Swiss infrastructure. Customer data never leaves the country. Uploads are token-protected, LLMs run in secure containers, and MongoDB stores extracted data with full compliance. 

In other words, the system isn’t just efficient — it’s legally deployable in real-world Swiss environments.

---

## What We Learned

We hit several roadblocks during development, both technical and operational.

Scanned documents were particularly tricky. OCR errors — especially in short keywords like “ja” or “nein” — reduced accuracy in boolean fields. Scaling LLM services made the biggest difference in performance. By contrast, scaling other parts of the pipeline, like Celery workers or NLP services, had limited effect.

Standardizing data formats across insurers was another challenge. Each provider structures its policies differently, and building a general schema that could accommodate all variations turned out to be harder than expected. 

We also learned that good monitoring isn’t optional. Tools like [Prometheus](https://prometheus.io/) or [Flower](https://flower.readthedocs.io/) would have saved us time by identifying pipeline bottlenecks earlier.

---

## What’s Next?

Here’s what we’re planning for the next phase:

- GPU acceleration for LLM inference  
- A better UI for reviewing extracted content  
- Automatic email and SMS delivery of upload links  
- Retry logic and dead-letter queues for failed jobs  
- Centralized logging and observability with [OpenTelemetry](https://opentelemetry.io/)

We’re also adding advanced monitoring, error tracking, and alerts. These improvements will make the system more reliable, scalable, and ready for production use.

---

> “If insurance data is the new oil, why are we still digging it up with shovels?”

---

## Want to Learn More?

We’ve documented our architecture, evaluation results, and development lessons in a full technical report.  
If you're part of a Swiss insurance provider — or simply curious about AI-powered document processing — [get in touch](mailto:schlesel@students.zhaw.ch). We’ll be happy to share the report with you.

<p align="center">
<img src="https://user-images.githubusercontent.com/74038190/226127923-0e8b7792-7b3c-462b-951b-63c96ba1a5af.gif" width="25" data-target="animated-image.originalImage">
</p>