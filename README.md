# Medical RAG Experiments

This repository contains exploratory experiments around **retrieval-augmented generation (RAG)** for medical question answering and **MedRAG-inspired differential diagnosis workflows**. The current work focuses on three threads:

1. **RAG benchmarking** on PubMedQA-style corpora
2. **Biomedical generative model benchmarking / prototyping** for diagnosis-oriented prompting
3. **Knowledge-guided reasoning RAG** as a planned next step for stronger MedRAG-style reasoning

> **Current status:** this is an experimental research repo, not a production system and not a clinical tool.

---

## What this repo currently explores

### 1) Retrieval benchmarking for medical RAG
The repo includes notebooks for comparing multiple retrieval approaches on PubMedQA-derived corpora:

- Dense retrieval with sentence-transformer embeddings
- Dense retrieval with **MedCPT** encoders
- Sparse retrieval with **BM25**
- Hybrid retrieval with **Reciprocal Rank Fusion (RRF)**
- Vector storage and search with **Qdrant**

The goal is to understand how retrieval quality changes as corpus size, encoder choice, and retrieval strategy change.

### 2) Generative biomedical model prototyping
The repo also includes early experiments with a small instruction model for diagnosis-oriented generation:

- Case-based differential diagnosis prompting
- Using retrieved similar cases as context
- Producing top diagnosis candidates + short reasoning + follow-up questions
- Simple iterative follow-up workflow

This part is currently more of a **prototype / qualitative benchmark scaffold** than a finalized quantitative model benchmark.

### 3) MedRAG-inspired diagnosis workflow
A separate notebook explores a **MedRAG-inspired** differential diagnosis setup on DDXPlus-like patient cases:

- retrieve similar historical cases
- generate a likely diagnosis and alternatives
- ask follow-up questions
- update the case and rerun reasoning
- expose the workflow through a simple Gradio UI

At the moment, this should be treated as a **partial / in-progress replication direction**, not a strict full replication of the MedRAG paper.

---

## Repository contents

### `BigRagExperiments.ipynb`
Main experimental notebook containing multiple stages:

- small end-to-end medical RAG baseline
- larger PubMedQA retrieval setup
- dense retrieval benchmarking
- BM25 vs dense vs hybrid comparison
- MedCPT-based larger-scale dense retrieval setup
- Colab-oriented experiments around the MedRAG codebase

### `MedRAG_bench_marking_Retriver_e_.ipynb`
Focused retrieval benchmarking notebook using cached corpora / embeddings and BM25 evaluation.

### `Medrag_Differntial_Diagnosis_System.ipynb`
Prototype notebook for differential diagnosis over DDXPlus-style cases with:

- BM25 retrieval over prior cases
- generation using a compact instruction model
- iterative follow-up flow
- Gradio demo

---

## Datasets and resources used

The current notebooks reference the following datasets / resources:

- **PubMedQA**-style corpora (`pqa_artificial`, `pqa_labeled`)
- **DDXPlus**-style patient case data (`aai530-group6/ddxplus`)
- **Qdrant** for vector indexing
- **MedCPT** retriever components for dense retrieval experiments
- **BM25** via `rank-bm25`

---

## Benchmark results

## 1) PubMedQA retrieval benchmark — smaller experimental setup

**Notebook:** `BigRagExperiments.ipynb`  
**Corpus size:** 20,500 documents  
**Evaluation queries:** 500  
**Dense encoder:** `sentence-transformers/all-MiniLM-L6-v2`

### Results

| Method | Queries | Accuracy@1 / Hit@1 | Hit@3 | Hit@5 | Hit@10 | MRR |
|---|---:|---:|---:|---:|---:|---:|
| Dense | 500 | 0.944 | 0.982 | 0.986 | 0.998 | 0.9647 |
| BM25 | 500 | 0.998 | 1.000 | 1.000 | 1.000 | 0.9990 |
| Hybrid RRF | 500 | 0.980 | 0.998 | 1.000 | 1.000 | 0.9888 |

### Observations

- In this setup, **BM25 strongly outperformed dense retrieval** at rank 1.
- Hybrid RRF improved over dense retrieval, but still did not surpass BM25 on top-1 accuracy.
- This suggests that the benchmark currently rewards strong lexical overlap and exact-term matching.

### Important interpretation note

These results are useful for **comparing retrieval behavior inside this experimental setup**, but they should not yet be treated as a definitive statement that BM25 is universally better than dense medical retrieval.

Likely reasons BM25 performs extremely well here include:

- strong lexical overlap between evaluation queries and gold documents
- the structure of the PubMedQA-derived corpus
- the way labeled / artificial subsets are mixed in the current experiments

---

## 2) PubMedQA retrieval benchmark — larger MedCPT setup

**Notebook:** `BigRagExperiments.ipynb`  
**Corpus size:** 51,000 documents  
**Evaluation queries:** 1,000  
**Dense retriever:** MedCPT-based dense retrieval + Qdrant

### Dense retrieval results

| Metric | Value |
|---|---:|
| Queries | 1000 |
| Accuracy@1 / Hit@1 | 0.924 |
| Hit@3 | 0.985 |
| Hit@5 | 0.993 |
| Hit@10 | 0.997 |
| MRR | 0.9539 |

### Setup notes

- Corpus embeddings were built for **51,000 documents**
- Embedding dimension: **768**
- Reported corpus encoding time in notebook: **1672.15 seconds**
- Retrieval was evaluated through Qdrant nearest-neighbor search

---

## 3) BM25 benchmark on cached larger corpus

**Notebook:** `MedRAG_bench_marking_Retriver_e_.ipynb`  
**Evaluation queries:** 1,000  
**Retriever:** BM25

### Results

| Metric | Value |
|---|---:|
| Queries | 1000 |
| Hit@1 | 0.999 |
| Hit@5 | 1.000 |
| Hit@10 | 1.000 |
| MRR | 0.9995 |

### Observation

Again, BM25 is extremely strong in the current benchmark formulation. This is a useful result, but it also indicates that the benchmark design should be stress-tested further before drawing broad conclusions about real-world medical retrieval.

---

## 4) Differential diagnosis prototype

**Notebook:** `Medrag_Differntial_Diagnosis_System.ipynb`

### Current setup

- Dataset subset loaded: **20,000 cases**
- Train split: **16,000**
- Eval split: **4,000**
- Retriever: **BM25 over prior case texts**
- Generator used in notebook: **Qwen/Qwen2.5-3B-Instruct**
- Demo interface: **Gradio**

### What the prototype currently demonstrates

- retrieval of top similar cases for a patient description
- generation of:
  - most likely diagnosis
  - alternative diagnoses
  - short reasoning
  - follow-up questions
- simple second-round reasoning after adding follow-up information

### Example qualitative behavior

In one demo case, the system retrieved two similar cases with diagnosis **Viral pharyngitis**, and the model predicted:

- **Most likely diagnosis:** Viral pharyngitis
- **Alternatives:** Acute laryngitis, Bronchitis, Sinusitis

A second iterative round with added follow-up information retained Viral pharyngitis as the leading diagnosis while surfacing additional pulmonary alternatives such as **Pulmonary embolism** and **Costochondritis**.

### Important limitation

This notebook currently demonstrates the workflow qualitatively, but it **does not yet report aggregate diagnosis accuracy metrics** such as top-1 diagnosis accuracy, top-3 diagnosis recall, calibration, or follow-up efficiency.

---

## Key takeaways so far

1. **Sparse retrieval is currently very strong in this setup.**  
   BM25 performs exceptionally well on both the smaller and larger PubMedQA experiments.

2. **Dense retrieval is still strong, especially at top-k.**  
   Even when rank-1 lags behind BM25, dense retrieval still achieves high hit rates by rank 5 or 10.

3. **Hybrid retrieval helps, but benchmark design matters.**  
   RRF improves over dense-only retrieval, but the underlying benchmark still appears favorable to lexical retrieval.

4. **The diagnosis pipeline direction is promising, but still early.**  
   The DDXPlus workflow already shows retrieval + reasoning + follow-up interaction, but it needs stronger evaluation and more structured medical reasoning.

---

## What this repo is **not** claiming yet

This repo currently does **not** claim to be:

- a production-ready medical RAG system
- a validated clinical decision support tool
- a complete benchmark suite for medical retrieval
- a strict full replication of the MedRAG paper

A more accurate description right now is:

> **RAG benchmarking experiments, biomedical generation prototypes, and an in-progress MedRAG-inspired differential diagnosis workflow.**

---

## Major limitations of the current work

### 1) Benchmark formulation likely favors lexical retrieval
The current PubMedQA-style benchmark appears to have strong overlap between query wording and gold document wording. That can make BM25 look unusually dominant.

### 2) Replication is partial, not full
The current diagnosis notebook is **inspired by MedRAG**, but it does not yet fully implement the paper's full knowledge-guided reasoning design.

### 3) Knowledge graph component is not integrated yet
A central future direction is adding a knowledge graph / structured medical reasoning layer.

### 4) Evidence representation is still raw / coded
The diagnosis notebook currently uses evidence identifiers like `E_201`, which are not ideal for human-readable reasoning or public demos.

### 5) Quantitative evaluation for diagnosis is incomplete
The retrieval part has metrics, but the diagnosis-generation part still needs strong aggregate evaluation.

### 6) Notebooks are still research notebooks
The current notebooks are exploratory and Colab-oriented. They still need cleanup for fully reproducible open-source use.

---

## Work to be done

## Near-term cleanup

- [ ] clear notebook outputs and reduce file size
- [ ] remove Colab-specific hardcoded paths where possible
- [ ] organize notebooks in a clearer sequence
- [ ] add a proper `requirements.txt`
- [ ] add `.env.example` for API / HF credentials
- [ ] modularize common utilities outside notebooks

## Retrieval benchmarking improvements

- [ ] redesign the evaluation so it is less biased toward lexical overlap
- [ ] add stronger baselines beyond BM25 and the current dense encoders
- [ ] compare more biomedical retrievers
- [ ] evaluate hybrid methods more systematically
- [ ] test chunking strategies and metadata-aware retrieval
- [ ] report latency, memory, and indexing tradeoffs

## Generative benchmarking improvements

- [ ] compare multiple biomedical / instruction-tuned models
- [ ] add structured evaluation for diagnosis quality
- [ ] measure top-1 / top-3 diagnosis accuracy
- [ ] evaluate reasoning faithfulness against retrieved evidence
- [ ] benchmark follow-up question usefulness

## MedRAG-style reasoning roadmap

- [ ] add a **medical knowledge graph** layer
- [ ] map raw evidence codes to human-readable symptoms / findings
- [ ] incorporate symptom-to-diagnosis and diagnosis-to-evidence relations
- [ ] support knowledge-guided reranking of retrieved cases / documents
- [ ] move from case similarity only to **knowledge-guided reasoning RAG**
- [ ] align more closely with MedRAG-style multi-step diagnostic reasoning

## Product / demo improvements

- [ ] make the Gradio demo easier to run locally
- [ ] add example patient cases
- [ ] improve explanation formatting
- [ ] show retrieved evidence with source attribution
- [ ] add iterative follow-up memory in the UI

---

## Suggested roadmap

### Phase 1 — cleanup and reproducibility
Turn the current notebooks into a more reproducible experimental repo.

### Phase 2 — stronger benchmarks
Improve evaluation design so retrieval comparisons are more reliable and less artifact-driven.

### Phase 3 — diagnosis evaluation
Add aggregate metrics for diagnosis quality, not just retrieval quality.

### Phase 4 — knowledge-guided reasoning
Integrate knowledge graphs and structured reasoning so the repo moves closer to a true MedRAG-style system.

---

## Setup notes

Because the current work was developed largely in notebook / Colab style, setup may require:

- Python 3.10+
- PyTorch
- Transformers
- Datasets
- Qdrant client
- Sentence Transformers
- rank-bm25
- pandas / numpy / tqdm
- optional Gradio for UI

A future version of this repo should include:

- `requirements.txt`
- `README` setup steps
- environment variable instructions
- dataset preparation instructions

---

## Safety note

This repository is for **research and engineering experimentation only**.
It is **not** a medical device, **not** a diagnostic authority, and **not** intended for real clinical decision-making.

---

## Current one-line description

**RAG benchmarking, biomedical generative model prototyping, and knowledge-guided reasoning RAG experiments for medical QA and differential diagnosis.**

---

## Future positioning

Once the knowledge graph layer and stronger evaluation are added, this repo can evolve from:

> exploratory notebooks for medical RAG

into:

> a more complete benchmark and MedRAG-style reasoning framework for medical question answering and differential diagnosis.

