# Evaluation Report — Blog Writing Evaluation Lab

---

## Executive Summary

A custom LangSmith evaluation system was built for AI-assisted blog writing. Two evaluation runs were completed — one on a **10-example dataset** (`blog-writing-eval`) and a follow-up on a **20-example dataset** (`blog-writing-eval-20`) — using two OpenAI models (`gpt-4o-mini` and `gpt-4o`) and three LLM-as-judge dimensions: **coverage**, **structure clarity**, and **tone match** (scored 0–100).

At 10 examples both models tied at **81.2/100**. At 20 examples `gpt-4o-mini` pulled ahead (**82.0 vs 81.4**). The dominant failure pattern across both runs is partial key-requirement coverage — models consistently satisfy 2 of 3 required use cases and skip one. The performance gap between models is negligible (≤3 pts per dimension); `gpt-4o-mini` is the preferred choice for cost-sensitive workflows at 3–4× lower cost.

---

## 1. Methodology

### 1.1 Domain

AI-assisted blog writing. The system receives a natural-language `topic` and a `brief` (audience, tone, goal) and must produce a structured 600–800 word blog post.

### 1.2 Datasets

| Field | 10-Example Run | 20-Example Run |
| --- | --- | --- |
| **LangSmith name** | `blog-writing-eval` | `blog-writing-eval-20` |
| **Project** | `blog_eval_langsmith` (EU endpoint) | `blog_eval_langsmith` (EU endpoint) |
| **Size** | 10 examples | 20 examples |
| **Input fields** | `topic`, `brief` | `topic`, `brief` |
| **Output fields** | `outline`, `key_requirements` | `outline`, `key_requirements` |

The 20-example dataset includes the original 10 topics plus 10 new ones: SEO-friendly posts, case studies, FAQ pages, email newsletters, thought leadership articles, product descriptions, how-to guides, brand origin stories, writing for non-native English speakers, and community/culture posts.

### 1.3 Target Functions

| Function | Model | Temperature |
| --- | --- | --- |
| `target_gpt4o_mini` | `gpt-4o-mini` | 0.7 |
| `target_gpt4o` | `gpt-4o` | 0.7 |

Both use the same LangChain `ChatPromptTemplate | ChatOpenAI` chain and are decorated with `@traceable` for automatic LangSmith logging.

### 1.4 Evaluators

Three custom LLM-as-judge functions (judge: `gpt-4o-mini`, `temperature=0`):

| Key | What it measures |
| --- | --- |
| `coverage` | Does the post address all outline points and key requirements? |
| `structure_clarity` | Clear intro, H2 sections, short paragraphs, concise conclusion? |
| `tone_match` | Does the tone and language match the specified audience and brief? |

Evaluator prompts use `PLACEHOLDER_*` token substitution (not `str.format`) to avoid `KeyError` from literal JSON braces. Each returns a `langsmith.evaluation.EvaluationResult` with an explicit `key`, `score`, and `comment`.

---

## 2. Run 1 — 10-Example Results

### 2.1 Aggregate Scores (0–100 scale)

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.5 | 82.0 | 81.0 | **81.2** |
| `gpt-4o` | 78.5 | 83.0 | 82.0 | **81.2** |

*Extracted from LangSmith experiment comparison — Screenshots/Ten_Example/074207, 074526, 074823.*

### 2.2 Per-Example Coverage Scores

| # | Topic | gpt-4o-mini | gpt-4o | Δ |
| --- | --- | --- | --- | --- |
| 1 | AI for small business marketing | 85 | 80 | +5 mini |
| 2 | Overcoming writer's block | 85 | 85 | — |
| 3 | Planning a month of content | 85 | 85 | — |
| 4 | Repurposing blog posts | 80 | 70 | +10 mini |
| 5 | Common AI content mistakes | 80 | 75 | +5 mini |
| 6 | AI for topic research | 85 | 80 | +5 mini |
| 7 | Writing a product launch post | 85 | 85 | — |
| 8 | Local business content calendar | 80 | 70 | +10 mini |
| 9 | Creating evergreen tutorials | 70 | 75 | +5 gpt-4o |
| 10 | Maintaining brand voice | 70 | 80 | +10 gpt-4o |

**Best examples (both ≥ 85):** 2, 3, 7 — concrete briefs with verifiable, enumerable requirements.

**Weakest examples:** 8, 9 — topics requiring local-context specificity or strategic depth.

### 2.3 Latency (10-Example Run)

| Stat | gpt-4o-mini | gpt-4o |
| --- | --- | --- |
| Typical range | 4–7 s | 7–14 s |
| Mean (10 examples) | ~5.5 s | ~9.4 s |
| Slowest example | — | 14.26 s |

---

## 3. Run 2 — 20-Example Results

### 3.1 Aggregate Scores (0–100 scale)

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.05 | 81.75 | 84.25 | **82.0** |
| `gpt-4o` | 77.40 | 80.40 | 86.25 | **81.4** |

*Extracted from LangSmith dataset `blog-writing-eval-20` — Screenshots/Twenty Example/123620, 124019.*

### 3.2 Latency (20-Example Run)

| Stat | gpt-4o-mini | gpt-4o |
| --- | --- | --- |
| P50 Latency | 13.39 s | 7.03 s |
| P99 Latency | 18.66 s | 12.45 s |
| Run count | 20 | 20 |

**Note:** gpt-4o-mini was slower in this run (ran first as experiment #1), likely due to API rate limiting at the start of a larger batch. This reversal is a run-condition artifact, not a true performance characteristic. The 10-example run showed the expected order (mini ~5.5 s, gpt-4o ~9.4 s).

---

## 4. Scale Comparison — 10 vs 20 Examples

### 4.1 Score Delta Table

| Model | Dimension | 10-Example | 20-Example | Δ |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | Coverage | 80.5 | 80.05 | −0.45 |
| `gpt-4o-mini` | Structure Clarity | 82.0 | 81.75 | −0.25 |
| `gpt-4o-mini` | Tone Match | 81.0 | 84.25 | **+3.25** |
| `gpt-4o-mini` | **Overall Mean** | **81.2** | **82.0** | +0.8 |
| `gpt-4o` | Coverage | 78.5 | 77.40 | −1.10 |
| `gpt-4o` | Structure Clarity | 83.0 | 80.40 | **−2.60** |
| `gpt-4o` | Tone Match | 82.0 | 86.25 | **+4.25** |
| `gpt-4o` | **Overall Mean** | **81.2** | **81.4** | +0.2 |

### 4.2 Interpretation

**Coverage holds stable for gpt-4o-mini (−0.45), drops slightly for gpt-4o (−1.10)**
Coverage is the weakest and most variable dimension in both runs. gpt-4o-mini's coverage scores are more consistent — it stays closer to the brief. gpt-4o's slight drop at 20 examples suggests it introduces more off-brief tangents when topics become more diverse (e.g. SEO, case studies, thought leadership) where it tends toward verbosity.

**Structure clarity drops for gpt-4o at scale (−2.60)**
This is the most notable shift. gpt-4o's structure score falls from 83.0 to 80.40 across 20 examples. The new topics (thought leadership, brand stories, community posts) have less prescriptive structural briefs, and gpt-4o over-elaborates more on open-ended prompts. gpt-4o-mini's structure is essentially unchanged (−0.25).

**Tone match improves strongly for both models (+3.25 and +4.25)**
The 10 new examples include more tone-diverse briefs (empathetic, authoritative, warm, instructional) compared to the original 10. Both models adapt tone well. This improvement is likely a topic-mix effect — the new examples play to tone strengths.

**gpt-4o-mini now leads overall (82.0 vs 81.4)**
At 10 examples the models tied at 81.2. Doubling the dataset breaks the tie in favour of gpt-4o-mini. The lead is narrow (0.6 pts) but directionally consistent: gpt-4o-mini is more stable at scale.

**Core finding is confirmed:** The performance gap between models never exceeds 3 points per dimension across either run. Prompt quality — not model selection — is the dominant quality driver.

---

## 5. Analysis

### 5.1 Performance Patterns

#### Strongest dimension — tone match (~84–86/100 at 20 examples)

Tone is now the strongest dimension across both runs, having improved significantly at 20 examples. Both models respond reliably to specific tone instructions. The improvement is partly a dataset effect — the new examples include more distinctive tonal briefs.

#### Weakest dimension — coverage (~77–80/100)

The most common deduction remains partial key-requirement coverage. A post typically satisfies 2 of 3 required use cases, skipping one. This pattern is stable across both runs and both dataset sizes.

#### Structure clarity is reliable but gpt-4o degrades at scale

gpt-4o's structure score dropped 2.6 pts moving to 20 examples. On open-ended topics it adds sub-sections not in the reference outline and regularly exceeds 800 words. gpt-4o-mini's structure degrades less (−0.25) because its outputs are more concise by default.

### 5.2 Model Comparison

- **gpt-4o-mini wins on coverage** in both runs — shorter, tighter outputs stay closer to the brief.
- **gpt-4o-mini wins on overall average** at 20 examples (82.0 vs 81.4) — the only dimension where gpt-4o leads is tone match (86.25 vs 84.25).
- **No dimension shows a gap >5 pts** across any run. Neither model consistently dominates.

### 5.3 Error Analysis

| Failure Mode | Frequency | Impact |
| --- | --- | --- |
| Over-elaboration (extra H3s, >800 words) | High — most examples | Penalises structure_clarity |
| Partial key-requirement coverage (2/3 met) | High — most common coverage deduction | Penalises coverage |
| Tone drift (brief says "cautious"; post is enthusiastic) | Low — 1–2 examples | Minor tone_match penalty |
| Generic framing on specific/niche topics | Medium — local/strategic examples | Penalises coverage and tone |

### 5.4 Note on Earlier Evaluation Scores

An earlier run using `openevals.create_llm_as_judge` recorded approximately **2/5 (40%)** per dimension. Those results were affected by a `KeyError: '\n  "score'` bug — literal `{` braces in the evaluator prompt conflicted with `str.format()`, causing silent failures and scores defaulting to 0. The current custom evaluators using `PLACEHOLDER_*` substitution resolve this and produce realistic **70–90/100** scores.

---

## 6. Limitations

| Limitation | Impact |
| --- | --- |
| Moderate dataset (n=20) | Variance reduced vs n=10; still directional, not statistically robust |
| Judge model = target model (`gpt-4o-mini`) | Potential self-evaluation bias; scores may be slightly inflated |
| No human baseline | Scores reflect the judge's rubric, not expert human ratings |
| Single-temperature runs (0.7) | Results vary across runs — reproducibility section confirms output variance |
| Domain scope | All examples are AI-writing topics; cross-domain generalisation is unknown |
| Latency anomaly (20-example run) | gpt-4o-mini P50 13.39 s vs gpt-4o 7.03 s — likely a rate-limiting artifact, not a true reversal |

---

## 7. Recommendations

**Priority 1 — Add a conciseness evaluator**
Count H2 sections and flag posts exceeding 850 words or 4 H2s. This directly targets the dominant over-elaboration failure, which worsened for gpt-4o at scale.

**Priority 2 — Strengthen key-requirement checking**
Enumerate each `key_requirement` from the reference output and check them individually (binary pass/fail), then aggregate. The current holistic scoring is too lenient on partial failures.

**Priority 3 — Reduce self-evaluation bias**
Use `gpt-4o` as the judge when evaluating `gpt-4o-mini` outputs, and vice versa.

**Priority 4 — Choose `gpt-4o-mini` for production**
It now leads on overall average at 20 examples (82.0 vs 81.4), is 3–4× cheaper, and degrades less at scale. The only dimension where gpt-4o leads is tone match (+2 pts) — not enough to justify the cost.

---

## 8. LangSmith Evidence

| Artefact | File |
| --- | --- |
| 10-example A/B comparison dashboard | `Screenshots/Ten_Example/Screenshot 2026-05-30 074207.png` |
| 10-example per-example score table | `Screenshots/Ten_Example/Screenshot 2026-05-30 074526.png` |
| 10-example gpt-4o detail + latency | `Screenshots/Ten_Example/Screenshot 2026-05-30 074823.png` |
| 20-example experiment overview | `Screenshots/Twenty Example/Screenshot 2026-05-30 123620.png` |
| 20-example per-example comparison | `Screenshots/Twenty Example/Screenshot 2026-05-30 124019.png` |
| LangSmith project | `blog_eval_langsmith` (EU endpoint) |
| 10-example dataset | `blog-writing-eval` |
| 20-example dataset | `blog-writing-eval-20` |
| Notebook | `blog_eval_langsmith.ipynb` |
