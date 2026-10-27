# Search Engine for Opera Music

A domain-specific search engine for opera music, built end to end: **web crawling (Apache Nutch) → indexing (Apache Solr) → link analysis (PageRank, HITS) → clustering and query expansion → Flask API → web UI**.

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white) ![Apache Solr](https://img.shields.io/badge/Apache_Solr-D9411E?logo=apachesolr&logoColor=white) ![Apache Nutch](https://img.shields.io/badge/Apache_Nutch-1.19-555555) ![Flask](https://img.shields.io/badge/Flask-000000?logo=flask&logoColor=white) ![NetworkX](https://img.shields.io/badge/NetworkX-2C5E8C) ![NLTK](https://img.shields.io/badge/NLTK-154F5B) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)

---

## Overview

- Crawled and indexed **120,000+ pages** of opera music content with Apache Nutch into Apache Solr for full-text retrieval.
- Re-ranks Solr results with **PageRank** and **HITS** (hub and authority) scores computed over the crawled web graph.
- Groups results with **flat** and **hierarchical (HAC)** clustering.
- Supports **query expansion** with three relevance-feedback methods: association, metric and scalar clusters.
- The web UI shows results next to Google and Bing for the same query so you can compare them.

## Architecture

```
 seed URLs ─▶ Apache Nutch ──crawl/parse──▶ Apache Solr (core: nutch)
                   │                              │
                   └─ linkdb dump ─▶ webgraph.py ─┤  PageRank / HITS scores ─▶ results/
                                                  ▼
 frontend (HTML/JS) ──GET /opera/v1?query=&type=──▶ Flask API (search_engine.py)
                                                  ├─ page_rank | hits             → re-rank by link scores
                                                  ├─ flat / single / complete HAC → cluster-grouped results
                                                  └─ association | metric | scalar → expanded query → Solr
```

## Repository layout

| Path | Purpose |
|---|---|
| `backend/search_engine.py` | Flask API at `GET /opera/v1?query=<q>&type=<method>`. Queries Solr and applies the selected ranking, clustering or query-expansion method. |
| `backend/relevance_models/webgraph.py` | Parses the Nutch linkdb dump, builds the link graph with NetworkX, and writes `hub_scores.txt`, `authorities_scores.txt` and `pageRank_scores.txt` to `results/`. |
| `backend/qe/association_cluster.py` | Query expansion using association clusters (term co-occurrence). |
| `backend/qe/metric_cluster.py` | Query expansion using metric clusters (term proximity). |
| `backend/qe/scalar_cluster.py` | Query expansion using scalar clusters (similarity of association vectors). |
| `frontend/` | Search UI (`index.html`, `script.js`, `style.css`) with method selection and side-by-side Google/Bing comparison. |

Supported `type` values: `page_rank`, `hits`, `flat_clustering`, `single_HAC_clustering`, `complete_HAC_clustering`, `association_qe`, `metric_qe`, `scalar_qe`.

---

## Getting started

### Prerequisites

- Python 3.8+
- Java 11 (for Nutch and Solr)
- [Apache Nutch 1.19](https://nutch.apache.org/download/)
- [Apache Solr 8.11.2](https://solr.apache.org/downloads.html), with a core named `nutch`

```bash
pip install -r requirements.txt
python -m nltk.downloader stopwords
```

### 1. Crawl with Nutch

From the `apache-nutch-1.19` directory, put your seed URLs in `urls/seed.txt`, then run:

```bash
bin/nutch inject crawl/crawldb urls
bin/crawl --num-threads 120 crawl 2            # adjust threads for your CPU
bin/nutch readlinkdb crawl/linkdb/ -dump dump/linkdb
```

### 2. Index into Solr

```bash
bin/solr start -force                          # from the Solr directory; check http://localhost:8983/
bin/nutch invertlinks crawl/linkdb -dir crawl/segments
bin/nutch index crawl/crawldb/ -linkdb crawl/linkdb -dir crawl/segments/ -filter -normalize -deleteGone
```

### 3. Compute link-analysis scores

```bash
cd backend/relevance_models
python webgraph.py     # writes PageRank / HITS scores to ../../results/
```

Set `LINKDB_DUMP` and `RESULTS_DIR` if your directory layout is different. Nutch's built-in `webgraph`, `linkrank` and `scoreupdater` commands can also be used to write PageRank back into the crawldb.

### 4. Run the API and the UI

```bash
cd backend
python search_engine.py        # serves http://127.0.0.1:5000/opera/v1
```

Open `frontend/index.html` in a browser, enter a query and choose a ranking, clustering or expansion method.

Example API call:

```bash
curl "http://127.0.0.1:5000/opera/v1?query=la%20traviata&type=page_rank"
```

The clustering modes read precomputed cluster assignments from `results/clustering_f_3.txt` (one `url,cluster_id` per line).

---

## Tech stack

**Python** · **Apache Nutch** · **Apache Solr** (`pysolr`) · **Flask** · **NetworkX** · **NLTK** · **NumPy** · **HTML / CSS / JavaScript (jQuery)**
