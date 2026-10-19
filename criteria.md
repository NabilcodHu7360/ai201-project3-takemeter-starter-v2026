# Acceptance Criteria

These targets were written before training.

## 1. Overall accuracy is at least 0.65 on the held-out set for all three seeds.

**Why this target:** Three labels make chance performance about 0.33, so 0.65
would be useful while leaving room for difficult boundary cases.

## 2. Macro F1 is at least 0.60 for all three seeds.

**Why this target:** Macro F1 gives every label equal weight and prevents a
good overall score from hiding one weak category.

## 3. Every label has an F1 of at least 0.50 for all three seeds.

**Why this target:** Each label has 60 to 70 examples, leaving only about 9 to
11 per label in the test split, so per-label scores will be less stable.

## 4. Every label makes up between 25% and 40% of `labels.csv`.

**Why this target:** This keeps the 200 examples balanced enough that the
model cannot perform well by mostly guessing one label.

## 5. The most-confident third is at least 0.10 more accurate than the least-confident third for all three seeds.

**Why this target:** Confidence is useful only if high-confidence predictions
are meaningfully more reliable than uncertain ones.
