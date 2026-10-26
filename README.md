# TakeMeter

## What This Does

TakeMeter classifies public Hacker News comments as `analysis`,
`reasoned_opinion`, or `reaction`. The goal is to separate posts that provide
checkable support from posts that explain a view and short immediate replies.

## Label Taxonomy

### `analysis`

Explains or corrects a claim using a specific fact, number, example, or source.

- "The 6502 can't actually address more than that. Unless you use bank
  switching, which historically a lot of 6502 machines did use."
- "Namecheap used to be a perfect company. It turns out it was sold to MBA,
  PE bros [source]. This explains the deterioration of services."

### `reasoned_opinion`

States a view and gives a reason or experience, without checkable support.

- "It's just more convenient to stay in the terminal where I already do a
  lot of other things. Switching between GUIs is more of a hassle."
- "Since AI came along, I've pretty much lost interest in writing code by
  hand... I can imagine something similar happening in fields like math."

### `reaction`

Gives a brief response, question, joke, thanks, or feeling without explaining
why.

- "Great work on this, had a blast!"
- "Who owns the trademark on that?"

### Hardest boundary

The hardest boundary is `analysis` versus `reasoned_opinion`. I use `analysis`
only when the post includes a checkable fact, number, example, or source;
personal experience or reasoning without that support is `reasoned_opinion`.

## The Dataset

The 200 comments came from the public Hacker News API. An automated pass
suggested labels, and every row is marked as pre-labeled in `note`. Counts are
70 `analysis`, 70 `reasoned_opinion`, and 60 `reaction`.

Three hard cases:

1. A commenter questioned how police found 91 kg of meth. It could be
   `analysis`, but I chose `reasoned_opinion` because it infers a conclusion
   without giving a source.
2. A commenter described losing a named domain and linked three old threads.
   It could be `reasoned_opinion`, but I chose `analysis` because the specific
   claim is supported with checkable links.
3. A commenter thanked someone and asked which side of a bearing needed
   support. It could be `reasoned_opinion`, but I chose `reaction` because it
   asks an immediate question without an argument.

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size 16, maximum length
128, and seed 42. I kept the defaults.

**Device:** CPU with PyTorch 2.14.1.

**Split:** 139 train, 31 validation, and 30 test. The test set contains 11
`analysis`, 10 `reasoned_opinion`, and 9 `reaction` posts.

**Results:** Accuracy 0.833 and macro F1 0.835. Per-label F1 was 0.870 for
`analysis`, 0.762 for `reasoned_opinion`, and 0.875 for `reaction`.

## How I Used AI

AI helped me tighten the filters in the CSV while keeping the disclosure in
every row.

For Part 2, AI helped me identify and understand the staff-labeling task.

See RUNNING.md for commands.

## Baseline vs. Trained

**Prediction before running:** The trained model will beat the baseline most on
`reaction` because those posts have a clear short-response pattern.

| Measure | Baseline | Trained |
|---|---:|---:|
| Accuracy | 0.500 | 0.833 |
| Macro F1 | 0.436 | 0.835 |
| F1 — `analysis` | 0.600 | 0.870 |
| F1 — `reasoned_opinion` | 0.526 | 0.762 |
| F1 — `reaction` | 0.182 | 0.875 |

The prediction was correct: `reaction` improved most, by 0.693 F1.

## Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---:|---:|---:|---:|---|
| Accuracy | ≥0.65 | 0.833 | 0.733 | 0.800 | MET |
| Macro F1 | ≥0.60 | 0.835 | 0.733 | 0.809 | MET |
| Lowest label F1 | ≥0.50 | 0.762 | 0.714 | 0.737 | MET |
| Label shares | 25–40% | 30–35% | 30–35% | 30–35% | MET |
| Confidence gap | ≥0.10 | 0.300 | 0.100 | 0.000 | MISSED |

| true \ predicted | analysis | reasoned_opinion | reaction |
|---|---:|---:|---:|
| **analysis** | 10 | 1 | 0 |
| **reasoned_opinion** | 2 | 8 | 0 |
| **reaction** | 0 | 2 | 7 |

The largest error was two `reasoned_opinion` posts called `analysis`.

## Verdicts and Diagnoses

Criteria 1–4 were met. Criterion 5 missed because seed 2024 scored 0.700 for
both the most- and least-confident thirds. With only 10 posts per third, the
model's confidence was not reliably calibrated. The 0.100 accuracy spread also
showed that results still depended on the split.

## Agreement Report

Agreement was 30/30 (100%). There were no disagreements to adjudicate. The
staff rules were clear, including the rule that a decorative statistic does
not turn a hot take into analysis.

## The Improvement

I increased training from 3 to 4 epochs. This is the only system change, chosen
to test whether slightly more training improves confidence and seed stability.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---:|---:|---:|---:|---|
| Accuracy | ≥0.65 | 0.833 | 0.800 | 0.800 | MET |
| Macro F1 | ≥0.60 | 0.838 | 0.804 | 0.810 | MET |
| Lowest label F1 | ≥0.50 | 0.783 | 0.750 | 0.727 | MET |
| Label shares | 25–40% | 30–35% | 30–35% | 30–35% | MET |
| Confidence gap | ≥0.10 | 0.500 | 0.100 | 0.100 | MET |

It helped: accuracy spread fell from 0.100 to 0.033, and the confidence target
changed from MISSED to MET.

## What's Still Broken

`analysis` and `reasoned_opinion` remain the hardest boundary; two analysis
posts were called reasoned opinions. The labels require checkable support, but
the model still sometimes treats detailed personal reasoning as evidence. I
stopped after the required single improvement.
