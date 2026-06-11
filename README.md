# LLM_RAG_Neuroradiology

Automated MRI protocoling in neuroradiology using large language models (LLMs) with and without retrieval-augmented generation (RAG).

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Language](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](#)
[![Paper](https://img.shields.io/badge/DOI-10.1007%2Fs11547--025--02040--9-blue.svg)](https://doi.org/10.1007/s11547-025-02040-9)

---

## Overview

MRI protocoling—the assignment of suitable imaging sequences and the decision on contrast medium administration based on a clinical question—is a routine but expertise-dependent task in radiology. This repository contains the analysis code for a line of work evaluating whether LLMs can perform neuroradiological MRI protocoling, and whether grounding them in institution-specific guidelines through RAG improves their accuracy relative to radiologist benchmarks.

Two pipelines are provided: one reproducing the original peer-reviewed study, and one extending it longitudinally to a newer generation of open-weight and proprietary models.

For each clinical question, the pipeline tasks a model with predicting:

- expanded abbreviations from the referral,
- a working diagnosis and three differential diagnoses,
- the required MRI sequences, selected from a fixed institutional list of standardized sequences, and
- whether contrast medium should be administered.

Predictions are compared against a neuroradiologist-established gold standard and against protocols selected independently by four radiologists.

## Repository contents

| File | Description |
| --- | --- |
| `Automated MRI Protocoling Paper.ipynb` | Reproduces the published study—comparison of an open-weight model (Llama 3.1 405B) and a proprietary model (GPT-4o), each with and without RAG, against four radiologist readers. |
| `Automated MRI Protocoling Promotion.ipynb` | Extends the study longitudinally to a newer model generation (Llama 4 Maverick, GPT-5.2, Claude Opus 4.6), adds an embedding-model comparison, and analyzes the evolution of model performance across years. |
| `requirements.txt` | Pinned Python dependencies. |
| `LICENSE` | MIT License. |

## Pipeline

Both notebooks follow the same structure:

1. **Data standardization** — Radiology Information System (RIS) reports are loaded, cleaned, and reordered; MRI devices are mapped to their magnetic flux density; the clinical question and imaging-procedure description are extracted by keyword-anchored regular expressions; and contrast medium administration is flagged. The standardized table is written to CSV.

2. **Protocol prediction** — Each model is prompted, in German, under a neuroradiologist persona to return a strictly formatted response (abbreviations, diagnosis and differentials, sequences, contrast medium). Open-weight models are served through the Replicate API; GPT models through the OpenAI API. The structured output is parsed back into tabular columns with regular expressions.

3. **Retrieval-augmented generation** — For the RAG variants, institution-specific protocol guidelines (PDF) are loaded and chunked with a `RecursiveCharacterTextSplitter` (chunk size 400–450, no overlap), embedded with OpenAI `text-embedding-3-large`, and indexed in a FAISS vector store. At inference time the clinical question is used to retrieve the top *k* = 8 guideline chunks, which are passed to the model alongside the prompt.

4. **Statistical analysis** — Token-based symmetric accuracy is computed for each pipeline. Bootstrap 95% confidence intervals quantify uncertainty; the Wilcoxon signed-rank test compares sequence accuracy and the McNemar test compares contrast-medium accuracy between paired pipelines, with multiplicity controlled by Bonferroni correction. Retrieval quality is evaluated by checking whether the ground-truth protocol name appears among the retrieved chunks, and the two embedding models (`text-embedding-ada-002` vs. `text-embedding-3-large`) are compared.

5. **Figures** — Vector-store and retrieval diagnostics, sequence-evaluation plots, RAG-versus-radiologist comparisons, and cross-model comparisons.

## Models compared

| Class | Published study | Longitudinal extension |
| --- | --- | --- |
| Open-weight | Llama 3.1 405B | Llama 4 Maverick |
| Proprietary | GPT-4o | GPT-5.2, Claude Opus 4.6 |

Each model is evaluated both with and without RAG, and all are benchmarked against four radiologist readers (two board-certified radiologists and two residents).

## Requirements

The code targets Python 3.10+. Core dependencies (see `requirements.txt` for pinned versions):

- `pandas`, `numpy`, `scipy`, `statsmodels`, `scikit-learn` — data handling and statistics
- `openai`, `replicate` — model inference
- `langchain-community`, `langchain-text-splitters`, `langchain-openai`, `faiss-cpu` — RAG and vector retrieval
- `python-dotenv` — environment management
- `matplotlib`, `seaborn` — figures

## Setup

```bash
git clone https://github.com/laranoellereiner/LLM_RAG_Neuroradiology.git
cd LLM_RAG_Neuroradiology
pip install -r requirements.txt
```

Provide API credentials through a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_key
REPLICATE_API_TOKEN=your_replicate_token
```

Then open either notebook and update the file paths at the top of each pipeline to point to your standardized report table and your guideline PDF.

## Data availability

The clinical reports and institutional protocol guidelines are **not included** in this repository. They contain patient-identifiable information and institution-specific material that cannot be shared under data-protection regulations (GDPR/DSGVO). File paths and domain-specific keywords in the notebooks are therefore placeholders, intended to be replaced with local equivalents. The code is released to document the methodology and to support reproducibility of the analysis, not to redistribute the underlying data.

## Ethics

The study was conducted in accordance with the latest version of the Declaration of Helsinki and approved by the ethics committee of Charité – Universitätsmedizin Berlin (No. EA4/062/20). The need for informed consent was waived owing to the retrospective design.

## Citation

If you use this code, please cite the associated publication:

> Reiner LN, Chelbi M, Fetscher L, Stöckel JC, Csapó-Schmidt C, Guseynova S, Al Mohamad F, Bressem KK, Nawabi J, Siebert E, Wattjes MP, Scheel M, Meddeb A. Automated MRI protocoling in neuroradiology in the era of large language models. *Radiol Med*. 2025;130(11):1472–1482. doi:10.1007/s11547-025-02040-9

```bibtex
@article{Reiner2025MRIprotocoling,
  title   = {Automated MRI protocoling in neuroradiology in the era of large language models},
  author  = {Reiner, Lara Noelle and Chelbi, Moudather and Fetscher, Leonard and St{\"o}ckel, Juliane C. and Csap{\'o}-Schmidt, Christoph and Guseynova, Shakhnaz and Al Mohamad, Fares and Bressem, Keno Kyrill and Nawabi, Jawed and Siebert, Eberhard and Wattjes, Mike P. and Scheel, Michael and Meddeb, Aymen},
  journal = {La radiologia medica},
  volume  = {130},
  number  = {11},
  pages   = {1472--1482},
  year    = {2025},
  doi     = {10.1007/s11547-025-02040-9}
}
```

## License

Released under the MIT License. See [`LICENSE`](LICENSE) for details.

Copyright © 2024 Lara Noelle Reiner.

## Contact

Lara Noelle Reiner — Department of Neuroradiology, Charité – Universitätsmedizin Berlin.
