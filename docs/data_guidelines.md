# Data Handling Guidelines


Before models, get intimate with the data. EDA for multimodal reasoning isn’t just “count rows and plot histograms”—it’s about verifying alignment, difficulty, and whether your steps truly teach the model how to think.

---

## Objectives and scope

- **Data integrity:** Validate each modality (text, image, table, audio) and their links.  
- **Alignment quality:** Ensure prompts, context, and targets are correctly paired and synchronized.  
- **Reasoning signal:** Confirm presence and quality of step-by-step annotations, constraints, and verifiable answers.  
- **Difficulty coverage:** Map an “easy → hard” spectrum to support curriculum learning.  
- **Bias and leakage:** Detect shortcuts, spurious cues, and train–test contamination.  
- **Deployability:** Check lengths, sizes, and formats that will fit your training serving stack.

---

## Core EDA across modalities

### Text (prompts, chain-of-thought, answers)
- **Length distributions:**  
  - Inspect token counts for prompts, CoT steps, and final answers; flag extremes.
- **Structure consistency:**  
  - **Markers:** Verify presence and order of step markers (e.g., “Step 1:”, “Reasoning:”); check masks if you plan latent CoT.  
- **Language quality:**  
  - **Noise:** Detect boilerplate, templates, copypasta; deduplicate near-identical items.  
- **Answer types:**  
  - **Normalization:** Standardize units, formats (e.g., numbers vs text), and whitespace for exact-match evaluation.

### Vision (images, diagrams)
- **Basic integrity:**  
  - **Metadata:** Resolution, aspect ratio, color channels; detect corrupt files and unusual dimensions.  
- **OCR readiness:**  
  - **Text-in-image:** Estimate text density; test an OCR pass to see if diagrams/labels are legible.  
- **Content checks:**  
  - **Saliency:** Quick heatmaps or object counts to ensure non-empty images; detect blank plots or watermarks that could mislead.

### Structured data (tables, charts)
- **Schema validation:**  
  - **Headers/types:** Enforce column types; catch missing headers or mixed-type cells.  
- **Stat sanity:**  
  - **Outliers:** Z-scores/IQR for numeric fields; canonicalize units/currencies.  
- **Rendering alignment:**  
  - **Chart → data:** If charts are paired with source tables, verify they match (sampling a subset).

### Audio (if present)
- **Signal quality:**  
  - **Duration/SNR:** Check waveform stats; flag silence-heavy clips.  
- **Transcripts:**  
  - **Alignment:** ASR timestamps vs ground-truth steps; spot drift or missing segments.

---

## Cross-modal alignment and synchronization

- **ID consistency:**  
  - **Joins:** Validate that each sample has consistent IDs across modalities; report unmatched joins.  
- **Temporal/positional alignment:**  
  - **Timestamps:** For video/audio + text, check that step descriptions align with time segments.  
- **Referential accuracy:**  
  - **Anchors:** Randomly sample items and confirm text references (“the red bar”, “node C”) exist visually/in the data.  
- **Context leakage:**  
  - **File names/metadata:** Ensure answers aren’t accidentally embedded (e.g., label “solution_42.png”).

---

## Reasoning signal quality

- **Step validity:**  
  - **Checks:** Each step should be logically connected; run simple validators (unit consistency, arithmetic verification, constraint satisfaction).  
- **Completeness:**  
  - **Coverage:** Confirm that multi-step annotations reach the final answer; flag truncated chains.  
- **Ambiguity:**  
  - **Multiple solutions:** Mark items with acceptable alternatives; provide canonicalization rules.  
- **Self-consistency potential:**  
  - **Redundancy:** Identify items suitable for multiple-sample voting; this helps later with decoding/evaluation.

---

## Difficulty calibration and curriculum

- **Heuristics:**  
  - **Features:** Number of entities, operations, steps, modality count; build a difficulty score.  
- **Empirical difficulty:**  
  - **Pilot evals:** Run a baseline model to get per-item accuracy and loss; correlate with heuristics.  
- **Buckets:**  
  - **Strata:** Create easy/medium/hard splits ensuring modality diversity in each bucket.  
- **Progression:**  
  - **Curriculum:** Design sequences that gradually increase complexity, mixing modalities intentionally.

---

## Bias, shortcuts, and data leakage

- **Artifacts:**  
  - **Spurious cues:** Detect template phrases, answer-first patterns, color-coded solutions, or positional biases (e.g., “the rightmost bar is always correct”).  
- **Class balance:**  
  - **Labels:** Balance yes/no, true/false, and numeric ranges; avoid degenerate distributions.  
- **Train–test contamination:**  
  - **Hashing:** Near-duplicate detection across splits (text similarity, image perceptual hashes).  
- **Demographic and content sensitivity:**  
  - **Audit:** If humans are in images/audio, review representation and sensitive attributes; document policies.

---

## Operational checks for training and serving

- **Length and size budgets:**  
  - **Tokens/context:** Ensure typical prompt + CoT fit your planned context window; compute worst-case KV cache memory.  
- **Sharding and streaming:**  
  - **Files:** Verify shard sizes for efficient loading; test streaming pipelines for multimodal batches.  
- **Serialization formats:**  
  - **Consistency:** Use stable schemas (e.g., JSONL with URIs for images/audio); validate with schemas.  
- **Versioning:**  
  - **Manifests:** Record dataset version, preprocessing configs, and hashes for reproducibility.

---

## Metrics, reports, and sanity dashboards

- **Coverage metrics:**  
  - **Modality presence:** Percent of samples with each modality; missing rates.  
- **Quality metrics:**  
  - **Validator pass rate:** Steps passing unit/math checks; OCR success rate; schema validation rate.  
- **Difficulty profile:**  
  - **Distributions:** Items per difficulty bucket; baseline accuracy per bucket.  
- **Leakage/bias flags:**  
  - **Counts:** Duplicates, spurious cues, class imbalance indicators.  
- **Readiness gates:**  
  - **Checklist:** Minimum pass thresholds to move to training (e.g., >98% alignment integrity, <1% duplicates).


