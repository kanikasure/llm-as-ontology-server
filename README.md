# llm-as-ontology-server

## Pipeline Overview

This repository contains a three-step pipeline for evaluating LLM performance on SNOMED CT ontology tasks:

1. **Step 1**: LLM querying with Set 1 prompts (A1-A7)
2. **Step 2**: LLM querying with Set 2 prompts (B1-B6)
3. **Step 3**: Ground truth extraction and accuracy calculation

## Folder layout

- **`SnomedCT_files/`** — At repo root; holds SNOMED CT RF2 files. Shared by all runs; keep outside the testing folders.
- **`testing_gpt/`** — Notebooks for testing with GPT (OpenAI). Run Step 1, 2, 3 from here.
- **`testing_claude/`** — Same notebooks for testing with Claude.
- **`testing_gemini/`** — Same notebooks for testing with Gemini.
- **`testing_deepseek/`** — Same notebooks for testing with DeepSeek.

Each testing folder contains: `step1_llm_set1.ipynb`, `step2_llm_set2.ipynb`, `step3_ground_truth_and_accuracy.ipynb`, and `concept_list_helper.ipynb`. Notebooks resolve `SnomedCT_files` one level up (repo root).

**API usage by folder:**

| Folder            | API / client              | Env var              | Pip package        |
|-------------------|---------------------------|----------------------|--------------------|
| `testing_gpt/`    | OpenAI (GPT)              | `OPENAI_API_KEY`     | `openai`           |
| `testing_claude/` | Anthropic (Claude)        | `ANTHROPIC_API_KEY`  | `anthropic`        |
| `testing_gemini/` | Google Gemini             | `GOOGLE_API_KEY`     | `google-generativeai` |
| `testing_deepseek/` | DeepSeek (OpenAI-compatible) | `DEEPSEEK_API_KEY` | `openai`           |

## Usage

From any testing folder (e.g. `testing_gpt/`), run the notebooks sequentially in order:

1. **Step 1**: Open and run `step1_llm_set1.ipynb`
   - Queries LLM with Set 1 prompts (A1-A7)
   - Creates timestamped run directory
   - Saves results to CSV

2. **Step 2**: Open and run `step2_llm_set2.ipynb`
   - Queries LLM with Set 2 prompts (B1-B6)
   - Automatically finds the most recent run directory from Step 1
   - Saves results to CSV

3. **Step 3**: Open and run `step3_ground_truth_and_accuracy.ipynb`
   - Extracts ground truth from SNOMED CT RF2 files
   - Compares predictions against ground truth
   - Calculates accuracy metrics and generates summary

Each notebook includes markdown explanations between code sections to help you understand what each part does.

## Configuration

Before running, make sure to:

1. Place SNOMED CT RF2 files in `SnomedCT_files/` at the repo root. The notebooks in each testing folder point to this path automatically.
2. Set the API key for the folder you use (see table above), e.g.:
   ```bash
   export OPENAI_API_KEY="..."      # testing_gpt
   export ANTHROPIC_API_KEY="..."   # testing_claude
   export GOOGLE_API_KEY="..."       # testing_gemini
   export DEEPSEEK_API_KEY="..."     # testing_deepseek
   ```
   Install the matching pip package: `openai`, `anthropic`, or `google-generativeai` as needed.
3. Update the `CONCEPT_TERMS` list in each script if you want to use different concepts

## Output Files

Outputs are written under the repo root in **`output/<llm>/`**, so they stay separate from `SnomedCT_files/`:

- **`output/gpt/run_001/`** when running from `testing_gpt/`
- **`output/claude/run_001/`** when running from `testing_claude/`
- **`output/gemini/run_001/`** when running from `testing_gemini/`
- **`output/deepseek/run_001/`** when running from `testing_deepseek/`

Each run directory contains:

- **Step 1**: `step1_llm_set1/set1_llm_output.csv`
- **Step 2**: `step2_llm_set2/set2_llm_output.csv`
- **Step 3**: `step3_snomed_ground_truth/snomed_ground_truth.csv`, `comparison_set1_set2_A4_A7_vs_ground_truth.csv`, etc.

## Notes

- Each script supports resuming: if interrupted, re-running will skip already processed concepts
- Step 2 and Step 3 automatically find the most recent run directory created by Step 1 (within that LLM’s `output/<llm>/` folder)
- All scripts include logging to `logs.txt` files in their respective step directories
- **TEST_MODE**: Set `TEST_MODE = True` in the Configuration cell to run on 5 concepts only (saves API cost). A fixed seed (`42`) is used so step1, step2, and step3 all use the **same** 5 concepts for consistent testing