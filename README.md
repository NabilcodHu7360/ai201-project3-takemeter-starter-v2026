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

- AI helped propose a first-pass label from visible features such as sources,
  reasons, and reply length. I tightened the filters and kept the disclosure
  in every CSV row.
- AI pressure-tested the hardest boundary. I changed the rule so a reasoned
  view needs a checkable fact, example, number, or source to count as
  `analysis`.

**Pre-labeling disclosure:** All 200 rows were automatically pre-labeled and
reviewed against the written definitions. No row is marked as cold.

See [RUNNING.md](RUNNING.md) for commands.
