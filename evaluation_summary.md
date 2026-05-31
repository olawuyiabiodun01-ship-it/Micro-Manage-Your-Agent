# Evaluation Summary

## Domain & Dataset

**Domain:** AI-assisted blog writing. The system receives a `topic` and a `brief` (audience, tone, goal) and must produce a structured 600–800 word blog post with clear headings, short paragraphs, and a voice that matches the brief.

**Datasets:**

| Dataset | Examples | Topics covered |
| --- | --- | --- |
| `blog-writing-eval` | 10 | Marketing, writer's block, content planning, repurposing, AI mistakes, research, product launches, local calendars, evergreen tutorials, brand voice |
| `blog-writing-eval-20` | 20 | All of the above + SEO posts, case studies, FAQ pages, email newsletters, thought leadership, product descriptions, how-to guides, brand stories, non-native English writing, community posts |

Each example has **inputs** (`topic` + `brief`) and **reference outputs** (`outline` + `key_requirements`).

---

## Methodology

**Target function:** `generate_blog` — a LangChain `ChatPromptTemplate | ChatOpenAI` chain with `@traceable`, tested at `gpt-4o-mini` and `gpt-4o` (both `temperature=0.7`).

**Evaluators:** Three custom LLM-as-judge functions (judge: `gpt-4o-mini`, `temperature=0`), each returning a score 0–100:

| Dimension | What it measures |
| --- | --- |
| `coverage` | Does the post address all outline points and key requirements? |
| `structure_clarity` | Clear intro, descriptive H2 sections, short paragraphs, concise conclusion? |
| `tone_match` | Does the tone and language match the specified audience and brief? |

**Infrastructure:** LangSmith EU endpoint, project `blog_eval_langsmith`, `max_concurrency=2`.

---

## Results

### 10-Example Run

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.5 | 82.0 | 81.0 | **81.2** |
| `gpt-4o` | 78.5 | 83.0 | 82.0 | **81.2** |

### 20-Example Run

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.05 | 81.75 | 84.25 | **82.0** |
| `gpt-4o` | 77.40 | 80.40 | 86.25 | **81.4** |

### Scale Comparison (10 → 20 Examples)

| Model | Coverage Δ | Structure Δ | Tone Δ | Avg Δ | Verdict |
| --- | --- | --- | --- | --- | --- |
| `gpt-4o-mini` | −0.45 | −0.25 | **+3.25** | +0.8 | Stable — slight improvement |
| `gpt-4o` | −1.10 | **−2.60** | **+4.25** | +0.2 | Structure degrades at scale |

---

## Key Findings

**gpt-4o-mini leads at 20 examples (82.0 vs 81.4)**
The models tied at 10 examples (81.2 each). Doubling the dataset breaks the tie in favour of gpt-4o-mini, which is more consistent across diverse topic types.

**Coverage remains the weakest dimension for both models (~77–80/100)**
The most common failure is partial key-requirement coverage: posts satisfy 2 of 3 required formats or use cases and skip one. This pattern is stable across both runs.

**Tone match is now the strongest dimension (~84–86/100 at 20 examples)**
Tone match improved significantly at 20 examples (+3.25 for mini, +4.25 for gpt-4o). The new topics have more distinctive tonal briefs (empathetic, authoritative, instructional) that both models handle well.

**gpt-4o structure degrades at scale (−2.60)**
gpt-4o's structure clarity dropped from 83.0 to 80.40 at 20 examples. On open-ended topics it over-elaborates — adding extra sub-sections and exceeding 800 words. gpt-4o-mini's structure is essentially unchanged (−0.25).

**The model gap is still negligible (≤3 pts per dimension)**
Prompt quality — not model selection — remains the primary quality driver. Neither model dominates consistently across all topics.

**Dominant failure pattern: over-elaboration**
Models introduce extra H3 sub-sections not in the reference outline and frequently exceed 800 words. This worsened for gpt-4o at 20 examples.

---

## Limitations

- **Moderate dataset (n=20):** Variance reduced vs n=10; results are directional but not statistically robust.
- **Self-evaluation bias:** The judge model (`gpt-4o-mini`) evaluates outputs from the same model family, which may inflate scores.
- **No human baseline:** Scores reflect the judge's rubric interpretation, not expert human ratings.
- **Single domain:** All examples are AI-writing topics; generalisability to other domains is unknown.
- **Latency anomaly (20-example run):** gpt-4o-mini ran slower than gpt-4o in the 20-example batch (P50: 13.39 s vs 7.03 s), likely due to rate limiting on the first experiment. The 10-example run showed the expected order.

---

## Recommendation

Use `gpt-4o-mini` for production. It leads on overall average at 20 examples, is 3–4× cheaper, degrades less at scale, and the only dimension where gpt-4o leads (tone match, +2 pts) does not justify the cost difference for human-reviewed content workflows.

As a next step, add a **conciseness evaluator** that counts H2 sections and flags posts exceeding 850 words. This targets the dominant over-elaboration failure — the most consistent quality gap across both models, both runs, and all 20 examples.
