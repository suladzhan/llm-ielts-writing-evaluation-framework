# IELTS Writing Evaluation — LLM Calibration Dataset

A structured dataset for configuring a large language model (LLM) to evaluate IELTS Academic Writing responses. Built for use as a Claude Project knowledge base, but compatible with any LLM that supports system-level context injection (GPT-4, Gemini, etc.).

---

## What's in this repo

| File | Description |
|---|---|
| `band-descriptors.md` | Official IELTS writing assessment criteria (Task 1 + Task 2), bands 1–9, all 4 criteria. Sourced from IELTS Guide for Teachers (UCLES/Cambridge). Also includes official band rounding rules. |
| `calibration-essays.md` | 8 scored sample essays, Part 1. Bands 5.0–7.0. |
| `calibration-essays-2.md` | 8 scored sample essays, Part 2. Includes 6 examiner model answers (~Band 7.5–8) and 2 official scored samples (Band 5.5, 6.0). |
| `calibration-essays-3.md` | 8 scored sample essays, Part 3. Includes 2 real student Band 7.5 responses. Bands 5.5–7.5. |
| `system-prompt.md` | Ready-to-use system prompt. Copy into your LLM's system/instruction field. |

**Total: 24 calibrated essays** across all major Task 1 visual types and Task 2 question types.

---

## Coverage

### Band range
`5.0 · 5.5 · 5.5 · 6.0 (×7) · 6.5 (×6) · 7.0 · 7.5 (×2 real student) · ~7.5–8 (×6 examiner models)`

### Task 1 visual types
Line graph · Two-line comparison graph · Map/plans (present vs future) · Process/cycle diagram · Pie chart · Pie chart + bar chart · Grouped bar chart · Horizontal bar chart · Table + two pie charts

### Task 2 question types
Discussion (both views + opinion) · Agree/disagree · To what extent · Positive or negative development · Advantages outweigh disadvantages · Why + positive/negative

---

## How to use

### Option A — Claude Project (recommended)

This is the setup this dataset was built for.

1. Create a new Project in [claude.ai](https://claude.ai).
2. Open **Project instructions** and paste the contents of `system-prompt.md`.
3. Upload all four `.md` files to **Project knowledge**:
   - `band-descriptors.md`
   - `calibration-essays.md`
   - `calibration-essays-2.md`
   - `calibration-essays-3.md`
4. Start a conversation in the project. To evaluate an essay, send:
   ```
   [Paste the task prompt]
   [Paste or describe the visual if Task 1]
   [Paste the student's essay]
   ```
   The model will return a structured evaluation with per-criterion band scores, reasoning, and prioritised feedback.

### Option B — API / Custom deployment

Inject the files as context at inference time.

```python
import anthropic

# Load your knowledge files
with open("band-descriptors.md") as f:
    descriptors = f.read()
with open("calibration-essays.md") as f:
    cal1 = f.read()
with open("calibration-essays-2.md") as f:
    cal2 = f.read()
with open("calibration-essays-3.md") as f:
    cal3 = f.read()
with open("system-prompt.md") as f:
    system = f.read()

# Build context block
knowledge = f"""
<band_descriptors>
{descriptors}
</band_descriptors>

<calibration_essays_part1>
{cal1}
</calibration_essays_part1>

<calibration_essays_part2>
{cal2}
</calibration_essays_part2>

<calibration_essays_part3>
{cal3}
</calibration_essays_part3>
"""

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2000,
    system=system + "\n\n" + knowledge,
    messages=[
        {
            "role": "user",
            "content": """
TASK: Some people think competition is good. Others think we should cooperate more. Discuss both views and give your own opinion.

ESSAY:
In my opinion competition at work can both be a good thing and a bad thing...
[paste full essay here]
"""
        }
    ]
)

print(response.content[0].text)
```

### Option C — OpenAI / other providers

Same approach as Option B. Pass `band-descriptors.md` and the calibration files in the system message or as assistant context turns. Works with GPT-4o, Gemini 1.5 Pro, or any model with a large enough context window (the full dataset is ~40k tokens).

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": system + "\n\n" + knowledge},
        {"role": "user", "content": "[task + essay here]"}
    ]
)
```

---

## Expected output format

The model returns a structured evaluation every time:

```
Task type: Task 2
Word count: ~280

Criterion scores:
- Task Response: Band 6 — [2-3 sentences of reasoning with evidence from the essay]
- Coherence & Cohesion: Band 5 — [reasoning]
- Lexical Resource: Band 6 — [reasoning]
- Grammatical Range & Accuracy: Band 6 — [reasoning]

Overall Band: 5.5

Calibration: Closest to Sample 2 (Band 6.0, competition/cooperation essay) but 
sits below it due to absent paragraphing and lack of specific examples.

Top 3 improvements:
1. [specific, actionable]
2. [specific, actionable]
3. [specific, actionable]
```

---

## How the calibration works

The model does not "learn" from these files in the machine-learning sense. Instead, they act as **in-context reference material** — the LLM reads the band descriptors and scored essays at inference time and uses them to anchor its judgement.

This matters because:

- **Band descriptors** define the scoring rubric precisely (what separates band 6 from 7 on each criterion).
- **Calibration essays** show the model concrete examples of what each band looks like in practice, including the specific errors that keep an essay at 6.0 vs 6.5 vs 7.0.
- **Examiner comments** embedded in each sample teach the model to reason the same way a human examiner does — identifying which criterion is the weakest link and why.

Without calibration essays, LLMs tend to inflate scores. With them, scoring is significantly more consistent and grounded.

---

## Accuracy expectations

| Band range | Expected accuracy |
|---|---|
| Band 5.0–5.5 | ±0.5 — reliable |
| Band 6.0–6.5 | ±0.5 — reliable |
| Band 7.0–7.5 | ±0.5 — generally reliable |
| Band 8.0+ | ±0.5–1.0 — less reliable; no real student Band 8 anchors in dataset |

These are estimates based on the calibration data available. The model is not a certified IELTS examiner. For high-stakes decisions, treat output as a strong first-pass evaluation that a teacher should review, not as a final score.

The primary use case this dataset was built for: **replacing routine essay checking in a tutoring context**, where the goal is fast, structured, criterion-level feedback rather than a legally binding score.

---

## Extending the dataset

To add more calibration essays, follow the format in any of the three calibration files:

```markdown
## SAMPLE N — Test X, Task Y — Official Band Z.Z

**Task:** [full task text]

**Visual (if Task 1):** [text description of the graph/chart/map/diagram]

**Essay (Band Z.Z, text as written):**

> [full essay text, errors preserved]

**Per-criterion:**
- Task Achievement / Task Response: ~N — [reasoning]
- Coherence & Cohesion: ~N — [reasoning]
- Lexical Resource: ~N — [reasoning]
- Grammatical Range & Accuracy: ~N — [reasoning]

**Why Z.Z:** [1-2 sentences explaining the overall band]
```

Priority gaps to fill: real student Band 8 responses (currently only examiner models cover this range).

---

## Dataset sources

- Band descriptors: *IELTS Guide for Teachers*, UCLES/Cambridge Assessment English, © 2019. Official published criteria.
- Sample essays and examiner comments: *Cambridge IELTS* official practice test books (Cambridge University Press). All essays reproduced for educational/research purposes.
- Per-criterion breakdowns: derived from published examiner comments; official sub-scores are not published by Cambridge — breakdowns are educator inferences grounded in the examiner commentary.

---

## License

The calibration framework, system prompt, and file structure are MIT licensed. The underlying IELTS band descriptors and sample essays remain the intellectual property of Cambridge Assessment English / UCLES. Use for educational and research purposes.
