# Benchmarks for reasoning models and how to evaluate them

Start with a focused set of benchmarks across math, commonsense, science, and multimodal reasoning. Use simple, reproducible metrics (exact match, accuracy) plus reasoning-aware measures (step correctness, self-consistency).

---

## Core text reasoning benchmarks

| Benchmark | Domain | Metric | Notes |
|---|---|---|---|
| GSM8K | Grade-school math word problems | Exact match | Evaluate with normalized numeric answers; CoT optional. |
| MATH | Competition-level math | Exact match | Harder than GSM8K; sanitize LaTeX and units. |
| ARC-Challenge | Science multiple choice | Accuracy | Use choice-letter accuracy; many distractors. |
| StrategyQA | Multi-hop commonsense | Accuracy | Short answers; tests implicit strategies. |
| CommonsenseQA | Commonsense multiple choice | Accuracy | Good for everyday reasoning. |
| TruthfulQA | Truthfulness under misleading prompts | Accuracy | Avoids parroting falsehoods. |
| DROP | Reading comprehension with discrete reasoning | Exact match/F1 | Requires arithmetic and coreference. |
| HotpotQA | Multi-hop QA | Exact match/F1 | Optionally measure evidence selection. |
| MultiArith / SVAMP / AQUA-RAT | Math + algebraic reasoning | Exact match | Useful diversity for math styles. |

---

## Code-as-reasoning (optional)

| Benchmark | Domain | Metric | Notes |
|---|---|---|---|
| HumanEval | Code generation | Pass@1/Pass@k | Proxy for precise step-by-step logic. |
| MBPP | Programming problems | Accuracy | Short tasks; good for structured reasoning. |

---

## Multimodal reasoning benchmarks

| Benchmark | Modality | Metric | Notes |
|---|---|---|---|
| ScienceQA | Image + text + science | Accuracy | Explains reasoning; good alignment checks. |
| MathVista | Math diagrams + text | Exact match | Visual math reasoning; strict formats. |
| MMMU | Broad academic multimodal | Accuracy | Diverse tasks; good coverage. |
| TextVQA | Image → text reading | Accuracy | Requires OCR-quality inputs. |
| DocVQA | Documents/tables → answers | EM/F1 | Table/diagram navigation. |
| ChartQA | Charts/plots → answers | Exact match | Verify chart-to-value grounding. |
| AI2D / TQA (diagram QA) | Diagrams + QA | Accuracy | Label grounding and references. |
| VQA-v2 / GQA | General visual QA | Accuracy | Less “reasoning” but useful sanity checks. |
| OK-VQA | External knowledge via images | Accuracy | Tests visual + commonsense knowledge. |
| WikiTableQuestions / TabFact | Tables | EM/Accuracy | Structured reasoning over tables. |

---

## How to evaluate: practical workflow

### 1. Standardize I/O and answer normalization
- **Text answers:** Strip whitespace, normalize numbers (e.g., “12”, “12.0”), canonicalize units (seconds vs s).
- **Multiple-choice:** Map to A/B/C/D indices; compare exact.
- **Multimodal:** Verify the sample has its image/table/doc present; fail fast on missing assets.

### 2. Implement baseline evaluators
- **Exact match (EM):** Compare normalized predicted vs gold answers.
- **F1 (token-level):** Useful for span extraction tasks (DROP, HotpotQA).
- **Accuracy:** For multiple choice and classification tasks.
- **Pass@k:** For code (HumanEval/MBPP) if you sample multiple candidates.

Example EM evaluator (text):
```python
def normalize(s):
    s = s.strip()
    s = s.replace(",", "")
    # number normalization
    try:
        return str(float(s))
    except:
        return s.lower()

def exact_match(pred, gold):
    return int(normalize(pred) == normalize(gold))
```

### 3. Reasoning-aware metrics
- **Step accuracy:** If you have step annotations, compare each predicted step against expected steps or run validators (unit checks, arithmetic verification).
- **Self-consistency win rate:** Sample k generations; majority-vote answer; report whether the vote matches gold.
- **Verifier score:** Use a rule-based or learned verifier to flag contradictions or constraint violations.

Self-consistency skeleton:
```python
def self_consistency(model, prompt, k=5):
    answers = [model.generate_answer(prompt) for _ in range(k)]
    vote = max(set(answers), key=answers.count)
    return vote, answers
```

### 4. Multimodal evaluation harness
- **Loader:** Reads JSONL with fields like {question, image_path, context, answer}.
- **Preprocessing:** OCR (if needed), image resizing, table parsing.
- **Batching:** One sample at a time for long contexts on 12GB GPU.
- **Metric:** Exact match or accuracy, plus per-modality breakdown.

### 5. Benchmark scripts and reports
- Create one script per benchmark, all sharing a common evaluation library.
- Log:
  - **Overall metric**
  - **By-difficulty buckets** (if available)
  - **Failure modes** (numerical precision, unit mismatches, missing assets)
- Save per-item outputs for error analysis.

Example CLI skeleton:
```bash
python eval_gsm8k.py --model-url http://localhost:8000/generate --split test --self-consistency 5
python eval_math.py --model-url http://localhost:8000/generate --latex-normalize --max-problems 1000
python eval_scienceqa.py --model-url http://localhost:8000/generate --with-images --ocr tesseract
```

---

## Decoding settings that affect scores

- **Deterministic tasks (math, tables):** temperature=0.0–0.2, top_p≈0.9, limit max_new_tokens to reduce rambling.
- **Open-ended QA:** temperature=0.7, top_p=0.95, consider self-consistency (k=3–7).
- **Code:** enable stop sequences; consider pass@k with k=5–10.

---

## Grounding and verification for multimodal tasks

- **Referential checks:** Ensure references like “the red bar” or “node C” exist; fail samples otherwise.
- **Numerical sanity:** For ChartQA/DocVQA, verify numeric answers against parsed data.
- **OCR confidence thresholds:** Exclude items where OCR fails or mark them for separate reporting.

---

## Avoiding leakage and ensuring fairness

- **Split hygiene:** Deduplicate across train/dev/test via text similarity and image perceptual hashes.
- **Contamination check:** If using public models, ensure your test sets aren’t trivially memorized (spot-check unexpected high scores).
- **Reproducibility:** Fix random seeds, log decoding params, and dataset versions.

---




