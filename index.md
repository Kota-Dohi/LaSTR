# LaSTR Project Page (Reproducibility Details)

This page provides reproducibility details that are omitted from the paper due to page limits.

Paper title: **LaSTR: Language-Driven Time-Series Segment Retrieval**

## 1. Model and API identifiers

- Segment captioning VLM: `gpt-5.2`
- VLM-as-a-judge (rating + binary): `gpt-5.2`
- API style: OpenAI Chat Completions API
- No variant suffix was used (no `-pro`, no `-thinking`).

## 2. Data snapshot used for paper results (0208)

Captioned H5 files:

- `outputs/segment_captioned_20260208/train.h5`
- `outputs/segment_captioned_20260208/val.h5`
- `outputs/segment_captioned_20260208/test.h5`

Observed counts:

- Train: 154,835 windows, 560,223 segment-caption pairs
- Val: 4,997 windows, 17,591 segment-caption pairs
- Test: 10,000 windows, 36,852 segment-caption pairs

## 3. Segment caption generation prompt (exact)

From `scripts/segment_contrastive/generate_segment_captions_full.py`:

```text
You are analyzing a time series plot.

Each highlighted region is a segment. The full time series is shown in the background.
For each numbered highlight, write ONE short description (10 words) that a human would say.

Requirements:
- Describe the segment's local shape (rising/falling/flat, smooth/noisy, steady/volatile).
- ALSO position it in the global context (e.g., rebound within a decline, pullback in an uptrend, continuation, possible reversal).
- Use qualitative language only.

Do NOT include:
- Any numbers or exact values
- Phrases like "segment 1" / "the first segment" / indices
- Any measurements

Return ONLY a JSON array with exactly {n_segments} strings, in order from 1 to {n_segments}.
```

### Caption generation API settings

From `scripts/segment_contrastive/generate_segment_captions_full.py`:

- `temperature = 1.0` when model name contains `gpt-5` (or `o1`)
- request `timeout = 120.0` seconds
- retries: up to 3 attempts
- response is accepted only when:
  - valid JSON
  - JSON type is list
  - list length equals expected `n_segments`

If parsing/validation fails after retries, the sample is treated as caption-generation failure.

## 4. VLM-as-a-judge prompts (exact)

From `scripts/eval/evaluate_retrieval_recall.py`.

### 4.1 Binary judge prompt

```text
You are evaluating whether the highlighted segment in the time series plot matches the query description. Answer YES if it matches, otherwise NO. Respond in JSON: {"match": true/false, "reason": "..."}.
```

### 4.2 Rating-5 judge prompt

```text
You are evaluating whether the highlighted segment in the time series plot matches the query description.

Give an integer score from 1 to 5:
1 = clearly does NOT match
2 = mostly does not match
3 = partially matches / ambiguous
4 = mostly matches
5 = clearly matches

Respond in JSON: {"score": 1-5, "reason": "..."}.
```

### Judge API settings

From `scripts/eval/evaluate_retrieval_recall.py`:

- `temperature = 0.0`
- token cap = `200`
  - for GPT-5 family: `max_completion_tokens=200`
  - otherwise: `max_tokens=200`
- parser expects JSON response; fenced blocks are stripped before parsing

## 5. Environment variables used by scripts

### Caption generation script

From `scripts/segment_contrastive/generate_segment_captions_full.py`:

- `API_KEY` (required)
- `API_ENDPOINT` (optional, default: `https://api.openai.com/v1`)

### Judge script

From `scripts/eval/evaluate_retrieval_recall.py`:

- API key: `API_KEY` or `OPENAI_API_KEY`
- base URL: `API_ENDPOINT` or `OPENAI_BASE_URL`

## 6. Main commands used in experiments

Detailed remote/local command flow is documented in:

- `docs/REMOTE_LOCAL_E2E_COMMANDS.md`

Core commands:

```bash
# (Remote) non-VLM retrieval evaluation
python scripts/eval/evaluate_retrieval_recall.py configs/eval/evaluate_retrieval_multi.yaml

# (Local) VLM judge only
python scripts/eval/evaluate_retrieval_recall.py configs/eval/evaluate_retrieval_recall_judge_only.yaml
```

## 7. Notes on CLIP baseline

CLIP model ID:

- `openai/clip-vit-base-patch32`

CLIP query template used in evaluation:

```text
The highlighted segment can be described as: {caption}
```

Candidate segment images are rendered with highlighted segment overlays (same style used by evaluation visualization code).

