# LaSTR Project Page (Reproducibility Details)

This page provides reproducibility details that are omitted from the paper due to page limits.

Paper title: **LaSTR: Language-Driven Time-Series Segment Retrieval**

## 1. Model and API identifiers

- Segment captioning VLM: `gpt-5.2`
- VLM-as-a-judge (rating + binary): `gpt-5.2`
- API style: OpenAI Chat Completions API
- No variant suffix was used (no `-pro`, no `-thinking`).

## 2. Data snapshot used for paper results
Observed counts:
- Train: 560,223 segment-caption pairs
- Val: 17,591 segment-caption pairs
- Test: 36,852 segment-caption pairs

## 3. Segment caption generation prompt (exact)

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

## 4. VLM-as-a-judge prompts (exact)

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

## 6. Notes on CLIP baseline

CLIP model ID:

- `openai/clip-vit-base-patch32`

CLIP query template used in evaluation:

```text
The highlighted segment can be described as: {caption}
```

Candidate segment images are rendered with highlighted segment overlays (same style used by evaluation visualization code).

