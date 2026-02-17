# llm-as-ontology-server

## Pipeline Overview

This repository contains a three-step pipeline for evaluating LLM performance on SNOMED CT ontology tasks:

1. **Step 1 — Ground Truth** (run once, shared): Validate concepts against BioPortal SNOMED CT API, replace NOT FOUND concepts, extract ground truth
2. **Step 2 — LLM Queries** (per LLM): Query the LLM with Set 1 (A1–A7) and Set 2 (B1–B7) prompts
3. **Step 3 — Accuracy** (per LLM): Compare LLM predictions against ground truth

## Folder layout

```
repo/
├── ground_truth/                    # Shared ground truth (run once)
│   └── step1_ground_truth.ipynb
├── testing_gpt/                     # GPT-specific notebooks
│   ├── step2_llm_queries.ipynb
│   ├── step3_accuracy.ipynb
│   └── concept_list_helper.ipynb
├── testing_claude/                  # Claude-specific notebooks
├── testing_gemini/                  # Gemini-specific notebooks
├── testing_deepseek/                # DeepSeek-specific notebooks
├── output/                          # All outputs (git-ignored)
│   ├── ground_truth/                # Shared GT (from step 1)
│   │   ├── ground_truth.csv
│   │   └── validated_concepts.csv
│   ├── gpt/run_001/                 # Per-LLM runs
│   │   ├── step2_llm_set1/
│   │   ├── step2_llm_set2/
│   │   └── step3_accuracy/
│   ├── claude/run_001/
│   ├── gemini/run_001/
│   └── deepseek/run_001/
└── scripts/                         # Build scripts for notebooks
```

**API usage by folder:**

| Folder            | API / client              | Env var              | Pip package        |
|-------------------|---------------------------|----------------------|--------------------|
| `ground_truth/`   | BioPortal API             | `BIOPORTAL_API_KEY`  | `requests`         |
| `testing_gpt/`    | OpenAI (GPT)              | `OPENAI_API_KEY`     | `openai`           |
| `testing_claude/` | Anthropic (Claude)        | `ANTHROPIC_API_KEY`  | `anthropic`        |
| `testing_gemini/` | Google Gemini             | `GOOGLE_API_KEY`     | `google-generativeai` |
| `testing_deepseek/` | DeepSeek (OpenAI-compatible) | `DEEPSEEK_API_KEY` | `openai`           |

## Usage

### 1. Extract Ground Truth (run once)

Open `ground_truth/step1_ground_truth.ipynb` and run all cells:
- Validates each concept against the BioPortal SNOMED CT API
- Replaces NOT FOUND concepts with backup SNOMED concepts
- Extracts ground truth (parents, grandparents, children, siblings)
- Saves to `output/ground_truth/`

### 2. Query LLMs (per testing folder)

From each testing folder (e.g. `testing_gpt/`), open `step2_llm_queries.ipynb`:
- Reads validated concepts from shared ground truth
- Queries the LLM with Set 1 (A1–A7) and Set 2 (B1–B7) prompts
- Writes `set1_llm_output.csv` and `set2_llm_output.csv`

### 3. Calculate Accuracy (per testing folder)

From the same folder, open `step3_accuracy.ipynb`:
- Reads shared ground truth + LLM predictions
- Calculates exact-match recall and Jaccard metrics
- Generates summary tables

## Prompt Sets

**Set 1 (A1–A7)** — Ontology-style prompts:
- A1: FSN, A2: Semantic tag, A3: Definition status
- A4: Parents, A5: Grandparents, A6: Children, A7: Siblings

**Set 2 (B1–B7)** — Natural language prompts:
- B1: Official name, B2: Kind, B3: Category type
- B4: Broader terms, B5: Grandparents, B6: Narrower terms, B7: Peers

## Configuration

Before running, set your API keys:

```bash
export BIOPORTAL_API_KEY="..."     # Required for Step 1
export OPENAI_API_KEY="..."        # testing_gpt
export ANTHROPIC_API_KEY="..."     # testing_claude
export GOOGLE_API_KEY="..."        # testing_gemini
export DEEPSEEK_API_KEY="..."      # testing_deepseek
```

Install the matching pip packages: `requests`, `openai`, `anthropic`, or `google-generativeai` as needed.

## Notes

- Ground truth is extracted **once** and shared by all LLMs — no redundant API calls
- Each notebook supports resuming: re-running skips already processed concepts
- Step 2 creates run directories; Step 3 automatically finds the latest run
- Ground truth stores **all** relationships (no truncation) for accurate evaluation
- **TEST_MODE**: Set `TEST_MODE = True` in Step 1 to use 5 random concepts (saves API cost)
