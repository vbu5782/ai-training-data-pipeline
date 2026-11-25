# AI Training Data Scraping: How Much Data You Really Need, How to Keep the Pipeline Clean, and What the EDPB's New Rules Require (Full Plan and Credit Comparison)

Most teams that need training data from the web end up with a few thousand deliberately chosen pages. The scale reference is worth keeping straight: Common Crawl's October 2026 archive, published on 21 October, holds 2.16 billion pages, 371 TiB of uncompressed content, drawn from 34.3 million registered domains. That volume is what a base model consumes. A fine-tuning set about one domain needs a few thousand pages you picked on purpose, which is a completely different engineering problem.

Your budget goes to cleaning, provenance and legal review. Request volume is the one line item you can predict in advance, and it is rarely the expensive one. What follows covers the sizing decision, a seven-step pipeline with the three places it usually breaks, where regex-based cleaning stops paying off, what the European Data Protection Board's Guidelines 03/2026 require you to be able to demonstrate, and how to read a scraping API's pricing before you commit.

> **Quick pick.** For a feasibility pilot, ScraperAPI's free plan gives you 1,000 API credits a month, with 5,000 credits during your first seven days and a maximum of 5 concurrent connections. For a recurring monthly refresh across a domain list, Startup costs **$149/month, or $134.10/month billed annually** ($1,609.20/year), with 1,000,000 credits and 50 concurrent threads. Both come with a 7-day no-questions-asked refund. 👉 [See ScraperAPI's plans and credit pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans side by side

ScraperAPI sells credits, not pages, and the credit cost of a request depends on what you are scraping and which features you switch on. The plan table below covers every tier the pricing page currently lists. Only two tiers publish a fixed monthly price; the rest are quoted on the page itself.

| Plan | API credits / month | Concurrent threads | Geotargeting | Price | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 (5,000 in first 7 days) | 5 | — | $0 | [Start on the free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20 | US & EU | See pricing page | [Check the Hobby tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | **$149/mo**, or $134.10/mo annual | [Check the Startup tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | See pricing page | [Check the Business tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | **$475/mo**, or $427.50/mo annual | [Check the Scaling tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level | See pricing page | [Check the Professional tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level | See pricing page | [Check the Advanced tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level | See pricing page | [Check the Enterprise tier](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Analytics history is capped at the last 30 days on Hobby and Startup, and unlimited from Business upward.

### What a page actually costs

This is the part that decides whether a plan fits your project. A standard HTML page costs 1 credit. Everything a training-data pipeline typically needs on hard targets costs more, and the add-ons stack.

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| E-commerce (Amazon, Walmart, eBay) | 5 |
| SERP (Google, Bing, including subdomains) | 25 |
| LinkedIn | 30 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX, Akamai) | +10 |
| JavaScript rendering (`render=true`) | +10 |
| Premium proxy (`premium=true`) | +10 |
| Screenshot | +10 |
| Ultra premium (`ultra_premium=true`) | 30 (paid plans only) |
| Ultra premium + JS rendering | 75 (paid plans only) |

Geotargeting, sessions, device type, output format and auto-parsing add nothing. Every response reports its own cost in the `sa-credit-cost` header, and you can price any URL before scraping it with the Domain Cost Estimator in the dashboard. A `max_cost` parameter caps individual requests.

Run the arithmetic on a realistic target and the headline number changes. A documentation site behind Cloudflare that also needs JavaScript rendering costs 1 + 10 + 10 = **21 credits per page**. On the Startup plan's 1,000,000 credits, that is roughly 47,000 pages a month rather than a million. Budget for that before you assume a mid-tier plan is enough. 👉 [Price your own target mix on ScraperAPI](https://www.scraperapi.com/pricing?fp_ref=coupons)

Running out mid-month behaves differently by tier. Lower plans can opt into auto-upgrade, which moves you to the next tier when credits run out; higher plans can enable pay-as-you-go at a fixed per-credit rate with a monthly spending cap you set yourself. Annual plans get the full year's credits upfront, can be used on any timeline, and can be renewed early for a fresh allocation.

## Start with the size of your dataset, not the size of the web

2.16 billion pages is a pretraining number. If you are fine-tuning a model on one domain, a few thousand carefully chosen pages will outperform a broad crawl, because the crawl brings template noise, near-duplicates and off-topic pages that you then pay to filter out.

Rough anchors for what different jobs need:

- **Supervised fine-tuning:** model providers accept files with as few as 10 examples; guides on the topic generally recommend starting with 50 well-written demonstrations and treating 100 to 1,000 as the range where behaviour stabilises.
- **Classification sets:** a few thousand balanced, well-labelled records often beat a much larger noisy set. Judge coverage and error types against a fixed validation set rather than chasing row count.
- **Continued pretraining or domain adaptation:** tens of billions of tokens. CuraWeb, a 2026 English corpus built from Common Crawl, reached 2 trillion tokens using n-gram plus semantic deduplication and quality filtering, and a 3B model trained on it scored 1.8% higher on average than its baselines.
- **Evaluation sets:** small, held out, and never used for tuning. A few hundred prompts with expected answers is normal.

Licensed corpora are a real alternative now. The Media & the Machine tracker counted 91 publicly announced AI content licensing deals by June 2026. If your budget allows, buying a licensed set for pretraining and spending your scraping effort on the narrow domain slice you actually care about is often the cheaper path.

## The seven-step pipeline, and the three places it breaks

Scope, source, collect, clean and dedupe, label, format as JSONL, validate. Most broken datasets come from a skipped step rather than a bad source.

1. **Scope.** Fix the task, the shape of one record, and the target row count before touching a crawler.
2. **Source.** Open datasets (Hugging Face Hub, Kaggle), crawl archives (Common Crawl), the live web, licensed providers, synthetic data. Pick per modality.
3. **Collect.** Seed-URL crawling with depth and page caps, site-specific extractors, single-page fetches for agent tools, or an official API where one exists and its rate limits fit your SLA.
4. **Clean and dedupe.** Strip navigation, footers and cookie banners. Hash normalised paragraphs for exact duplicates, then compare word shingles or embeddings for near-duplicates.
5. **Label.** Write the labelling guideline against the same schema the model will see at inference.
6. **Format.** One JSON object per line, with `url`, `retrieved_at`, `title` and `source` on every row so you can trace any example back to its page and rerun the build later.
7. **Validate.** Check structure, grounding, class balance and leakage.

Three failures account for most of the damage.

**Splitting by row instead of by source website.** Near-duplicate pages from the same site land in both the training and validation files, and your evaluation score goes up without the model getting better. Split on the domain.

**Trusting model-written labels.** Every label needs checking against a sentence on its source page, and the check should be code, not a prompt you hope behaves. User researcher Caitlin Sullivan, writing in Lenny's Newsletter, reported that across more than 2,000 hours of testing AI analysis workflows, every major model produced fabricated quotes and stitched-together "Frankenstein" quotes from multiple sources. Length constraints like "12 words or fewer" caused the stitching almost every time; requiring complete sentences fixed it.

**Leaving the boilerplate in.** Navigation and cookie chrome teach the model templates, and near-duplicates inflate metrics while burning compute. Language detection and Unicode normalisation belong in the same pass, along with dropping empty and near-empty pages.

One working habit before you scale: crawl a single domain, set a low page cap, pull 50 random rows and read them. Fix what you find there, then raise the limit.

## Where rule-based cleaning stops paying off

Hand-written heuristics are the default stack. A heuristic extractor such as resiliparse, trafilatura or jusText pulls main content out of the HTML, then dozens of regex filters clean it. The extractors disagree across page layouts, and taking the union of several recovers considerably more usable text than the best single one, which tells you how much the rules depend on the page's structure.

The filters have a measurable error rate. An independent LLM-as-a-judge evaluation found that **31% to 65% of the pages a given rule discarded were actually worth keeping**. Rules that catch one kind of junk also throw away good content, and the loss compounds across a filter chain.

ReScraper, a 0.6B-parameter model, replaces the whole stack with a single learned step trained to extract main content and then choose among four actions: keep the page as extracted, edit out noisy lines and spans, delete the page, or rewrite it when the writing is poor but the information is useful. Evaluated on 18.0 million Common Crawl pages, pretraining 400M, 1.4B and 2.8B models on its output raised the DCLM Core score by a relative **3.8% to 4.7%** over the strongest baseline at each scale (+4.6% at 1.4B, +3.8% at 2.8B). At 1.4B it beat the multi-agent curation pipeline DataOrchestra by a relative 6.6%. DCLM Core here means centred accuracy averaged over 22 downstream tasks covering commonsense reasoning, language understanding, reading comprehension, symbolic problem solving and world knowledge.

For decision-making, the crossover matters more than the benchmark. At a few thousand pages, read your data and write rules; a learned cleaner is overhead you cannot evaluate. Past a few million pages, the 31% to 65% discard rate becomes the dominant cost of your corpus, and a small learned refiner is worth the setup. Between those points, run both on the same 100,000 pages and compare usable tokens per source page.

## What the EDPB's 2026 guidelines change

On 7 July 2026 the EDPB adopted Guidelines 03/2026 on web scraping in the context of generative AI, version 1.0, now open for public consultation. They cover scraping from internet sources external to your organisation, whether you do it yourself, contract someone else, or buy an already-scraped dataset from a broker. Nothing here is legal advice, and the guidelines are a draft under consultation rather than settled rules.

Two distinctions drive most of the analysis. **Targeted scraping** collects against defined criteria such as domain, language or topic. **Untargeted scraping** follows links without collection criteria and may sweep a large share of the internet, which means the controller knows far less about which personal data it actually holds.

GDPR applies to the processing involved in scraping: extraction, cleaning, structuring, storage. In a mixed dataset, the personal-data portion stays in scope, and the controller carries the Article 5(2) accountability burden of demonstrating compliance. The EDPB also names memorisation and regurgitation of personal data from training data as a risk.

The legitimate-interests balancing test under Article 6(1)(f) is where scraping projects get uncomfortable. Factors the EDPB weighs:

- Data subjects' interest in self-determination over data they published for a different purpose. Large-scale indiscriminate collection can create a **chilling effect** on expression through self-censorship, and deepfakes add to the impact on dignity.
- Sensitivity. Scraping can sweep up special-category data, criminal data, location and financial detail, and data about **minors**. The more sensitive, the more weight against you.
- Practical asymmetry. Data subjects find it hard to object to scraping or make themselves unreachable, and current technology does not let you easily delete personal data from a trained model.
- **Reasonable expectations.** Fulfilling the Article 12–14 information duties is not enough on its own. Relevant factors include the nature of the source site, whether the person published the data themselves, and any anti-scraping measures on the site, such as **robots.txt, ai.txt or CAPTCHAs**.

Mitigating measures the EDPB lists as non-exhaustive: excluding sites only reachable behind a login, excluding specific data categories or high-risk publications, limiting collection by time period, publishing updated lists of scraped sites, offering a prior unconditional objection mechanism or opt-out list, deleting or anonymising unnecessary personal data, and deploying technical measures against memorisation.

Where special-category data is collected incidentally rather than intentionally, the EDPB suggests the CJEU's reasoning in *GC and Others* (C-136/17, 24 September 2019) on search engines may potentially apply, with Article 9(1) prohibitions operating within the framework of the controller's responsibilities, powers and capabilities. That is a possible line of argument, not a safe harbour.

One design point that follows directly: retrieval-augmented generation and weight updates carry different risk profiles. A RAG system pulls text at query time and can drop a source; a fine-tuned model has absorbed it. Teams commonly keep pretraining on first-party, licensed and clearly open sources, and treat third-party pages more cautiously.

## Site owners are starting to identify scrapers by name

User-Agent strings are a request standard, not an enforcement mechanism. Providers under-disclose their crawlers, community databases contain attribution errors, and spoofing is documented, including among well-known AI companies. Any identification strategy built on self-reported UA strings inherits those gaps.

A 2026 paper takes a different route: stand up a dynamic site, hand every visiting crawler a **unique canary token**, then ask AI chatbots about that site. If a chatbot's answer reliably contains one specific crawler's token, that chatbot has been fed by that crawler. Across **18 production chatbots**, the method identified which scrapers feed which assistants, including crawlers the companies had not publicly disclosed.

The paper also lays out the two paths by which a chatbot reaches web content: a pre-built index corpus, either from a proprietary pipeline or bought from a search provider, retrieved at inference time via RAG; or a live single-page fetch at query time. Most prior work on disrupting AI scraping targeted pretraining data, leaving real-time retrieval largely unstudied. The excerpt available stops before the paper's findings on whether going offline or setting robots.txt actually keeps a site out of AI output, so treat canary tokens as a detection method rather than a proven opt-out.

The practical takeaway for anyone building a pipeline: be identifiable, keep a record of what you fetched and when, and respect the signals a site publishes. Changing your User-Agent to slip past a block is both unreliable and the kind of choice that reads badly later.

## FAQ

**Does RAG carry the same legal exposure as training on the same pages?**
No, and the difference is operational as much as legal. A retrieval system can remove a source from its index; a trained model cannot easily forget it. The scraping itself still needs a lawful basis, and fine-tuning on scraped content still needs clear licensing.

**How many labelled examples does a classification task need?**
A few thousand balanced, carefully labelled records is often enough. Measure coverage and error types on a fixed validation set instead of adding rows.

**Can I scrape images and image-text pairs?**
Technically yes, and most crawlers will return image URLs or files. Copyright and personality rights are sharper for images than for text, and separate clearance is usually needed before commercial vision training. Keep image pipelines apart from text pipelines so a text RAG job does not inherit image CDN traffic.

**Why is social and UGC data the riskiest category?**
Terms of service and privacy review are strictest there, the content frequently includes minors, special-category details and directly identifying information, and the people in it never expected model training as a use.

**What is a sane first project?**
One domain, a low page cap, 50 rows read by hand. You will learn more about your data quality in an afternoon than from a month of crawling.

**Do robots.txt and ai.txt matter if I am outside the EU?**
They matter as evidence of what a site owner expects, and they are named explicitly in the EDPB's reasonable-expectations analysis. Ignoring them also affects terms of service and how quickly you get blocked.

## Choosing a plan, and two things to confirm before you pay

Match the plan to the job rather than to the credit number.

- **Testing whether a source is worth using at all:** the free plan, one domain, a low page cap, 50 rows read. 5,000 credits in the first week is enough to see what the extraction actually returns.
- **A recurring monthly refresh over a domain list, mostly standard pages:** Startup at $149/month, or $134.10/month billed annually, gives 1M credits and 50 threads. At 1 credit per page that is a million pages; at 21 credits for a Cloudflare-protected, JavaScript-rendered target it is around 47,000.
- **Mixed targets with anti-bot protection and rendering, refreshed continuously:** Scaling at $475/month ($427.50/month annual) gives 5M credits, 200 threads, country-level geotargeting and pay-as-you-go past the cap.
- **Large pretraining-scale collection:** the upper tiers, where concurrency reaches 500 threads and credits run into the tens of millions. Confirm the per-request cost mix with the Domain Cost Estimator before choosing a tier, because at this volume the add-on multipliers decide the bill.

Before you enter a card, confirm two things. How credits are consumed for your specific target mix, including bypass and rendering add-ons, and how overage is handled on your tier, since auto-upgrade and pay-as-you-go behave differently and only one of them keeps your monthly spend fixed. Both answers are on the pricing page, and the 7-day refund gives you a window to test the real numbers against your own URLs. 👉 [Compare ScraperAPI plans and start with the free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)
