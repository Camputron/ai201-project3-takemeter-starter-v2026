# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter sorts comments from **Hacker News** (news.ycombinator.com), a public
tech and startup discussion forum, by **what kind of support a comment offers
for what it says**. There are three labels. `evidence` comments point at
something a reader can check. `anecdote` comments rest on the poster's own
experience. `unsupported` comments are opinion, jokes, questions and reactions
with nothing behind them. On HN the comments that read as substantial are
nearly always the ones that link, cite, or explain a mechanism. That's the
distinction the classifier is trying to learn.

**Stretch feature attempted:** a second training run with one hyperparameter
changed (epochs 3 → 10). See *The Training Run*.

---

## Label Taxonomy

Candidate distinctions noted while reading (Milestone 1): some comments include
a link or a number; some begin "I've used / at my last job / in my
experience"; many are one-line quips or sarcasm; some are questions.
Questions and quips didn't hold up as labels of their own. Too few of them were
clean, and they offer no support either way, so they're folded into
`unsupported`.

### `evidence`

**Definition:** Backs its point with something a reader can check without
trusting the poster: a link, a specific number or date, a named source or
quote, or a concrete technical mechanism or instruction.

**Example 1:**
> The Pixel 5 isn't supported anymore. The Pixel 6a still has a little less than 2 years of support left. These have become pretty cheap.
>
> The Pixel 8 and 8a aren't that expensive either. And keep in mind that they are supported until 2030 and 2031 respectively. [1] They not only receive security updates for 7 years, instead of the 5 years for previous Pixel generations, but also have stronger hardware security, by implementing the ARM memory tagging extension. [2]
>
> [1] https://grapheneos.org/faq#device-lifetime
>
> [2] https://grapheneos.org/faq#recommended-devices

**Example 2:**
> It's not anymore: "in April 2026 the implementation of the agreement was put on an indefinite hold due to opposition from US President Donald Trump." So whatever ICANN was going to do with .io, it's all on hold for now.

### `anecdote`

**Definition:** Backs its point with the poster's own first-hand experience
(what they did, used, saw, or counted themselves), and offers nothing
checkable beyond that.

**Example 1:**
> I've grown increase hate towards Finder to the point that I avoid using at all costs. I've been migrating to the terminal, using fzf to find files and directories and yazi for a more graphical experience.
>
> How can it be called FINDER, if it can't FIND things? cmd+shift+g should be a fuzzy search, but it returns nothing 80% of the time. cmd+f often can't see files that are in first level folders inside my home folder.
>
> Meanwhile, hitting Esc+C in the terminal (via fzf) it's totally effective.

**Example 2:**
> It's been some years now, but apple maps put me into a loop once in Branson, Missouri.
>
> It drove me around a couple miles that went right back to the intersection where we started, and then wanted me to start the loop again.

### `unsupported`

**Definition:** Gives an opinion, judgment, joke, question, or reaction with no
support offered.

**Example 1:**
> This is not "coordinated vulnerability disclosure". It's waiting for slow vendors at the risk of everybody else.

**Example 2:**
> 22k is nothing on the internet.
>
> A random youtube channel have way more than that.

### The hardest boundary

**Which two labels:** `evidence` vs `anecdote`. Many HN comments tell a personal
story *and* include a link, a price, or a technical detail.

**The decision rule I used every time:**
If the post contains a link, a published number or date, a named source, or a
concrete technical mechanism that a reader could check without taking the
poster's word for it, it's `evidence`, even if it's wrapped in a personal
story. Numbers and observations the poster produced themselves ("I scrolled
and 40–60% were anonymous", "each cover costs me about 6 cents") don't count
as checkable. If that's all the support there is, it's `anecdote`.

A secondary rule for `evidence` vs `unsupported`: a factual-sounding claim
only counts as evidence if it's specific enough to look up (named, numbered,
dated, or a mechanism). A general claim about the world ("crawler bots bring
down any website") is `unsupported`.

---

## The Dataset

**Where the posts came from:** 360 public Hacker News comments, nine
from each of 40 high-traffic stories (250+ comments each, late 2025 to 2026),
pulled through the public HN Algolia search API
(`hn.algolia.com/api/v1/search?tags=comment,story_<id>`). I kept comments
between 60 and 700 characters and dropped exact duplicates. Two were dropped
during labelling: one was a bare link with no text, and the other was a joke
whose only content was a link, which the decision rule would have forced into
`evidence` for the wrong reason. **358 rows remain**, kept whole and not
trimmed.

**How I labelled them:** **All 358 rows were labelled by an AI (Claude, via
Claude Code)**, applying the definitions and decision rule above. That was done
with my go-ahead. Every row's `note` column says so.

- **338 rows** say `pre-labelled by Claude (AI); needs human review`.
- **20 rows are marked `cold`.** They were picked at random and relabelled by
  Claude in a second, unaided pass, working from the text alone without seeing
  the first-pass label. **They were not labelled by a human.** The second pass
  agreed with the first on 19 of 20 (95%). The one change was a short post
  about loving the field ("Yes, seen it all… I wish I'd have discovered the
  field earlier"), which moved from `unsupported` to `anecdote` because the
  poster is speaking from their own time in the field. **The 95% overstates
  real consistency:** the same model labelled both times, so it shares its own
  blind spots. Comparing against the staff set in unit 6 is a fairer test.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `unsupported` | 236 | 65.9% |
| `evidence` | 61 | 17.0% |
| `anecdote` | 61 | 17.0% |
| **Total** | 358 | 100% |

No label is above the 70% cap, but `unsupported` is close to it and the
notebook flags it as lopsided. That's how HN actually is: most replies are
opinion. It's also why the default training run collapsed (see below).

**Three hard cases** (all flagged `hard case:` in the `note` column)

**1.**
> *The post:* "My girlfriend's Facebook got stolen via a novel technique a few years ago: https://www.reddit.com/r/facebook/comments/14nbp1a/… Once the hacker got in, they enabled PGP with a random key to prevent the account recovery process from working. It took many, many months to get the account back…"
>
> *Could have been:* `anecdote` or `evidence`
>
> *I chose, because:* `evidence`. It's a first-hand story, but the link documents the exploit independently of the poster, and under the decision rule a checkable fact beats a personal frame.

**2.**
> *The post:* "Allowing anonymous signatories only weakens the petition. Two important people signing a petition is worth more than 10000 anons. I scrolled through a few pages and 40-60% are anonymous. …"
>
> *Could have been:* `evidence` or `anecdote`
>
> *I chose, because:* `anecdote`. It has a number, but the poster produced it by scrolling. Nothing outside the poster backs it, so it's an observation rather than a published figure.

**3.**
> *The post:* "Before the Civil War the majority of states had already banned the practice. It had to break down in order to be able to jam in an amendment to the constitution against it. …"
>
> *Could have been:* `evidence` or `unsupported`
>
> *I chose, because:* `evidence`. No source is given, but the claim is specific and historical, so it could be looked up, which the secondary rule accepts. This is the boundary I'm least sure about: change one word to "most places" and it would be `unsupported`.

Other flagged hard cases in the CSV include a price-plus-opinion laptop
comparison (`evidence`, because the price is published), a "6 cents per cover"
usage report (`anecdote`, because the number comes from the poster's own use),
and "I used a tractor like this" plus how a governor works (`evidence`, because
of the mechanism).

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Device:** `mps` (Apple Silicon GPU), torch 2.14.1, recorded in `results.json`.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128,
seed 42. These are the notebook defaults, unchanged for the run in
`results.json`.

**Anything I changed from the defaults, and why:** Nothing for the main run.
**Stretch:** a second run identical except `EPOCHS = 10`, saved separately as
`results_epochs10.json` so it doesn't overwrite the main result. The reason:
with 250 training rows and batch 16, three epochs is only 48 optimizer steps,
and the main run never moved off the majority label. It predicted
`unsupported` for all 54 test posts (accuracy 0.667, macro F1 0.267, F1 0.00 for
`evidence` and `anecdote`).

What moved with 10 epochs (same split, same seed): accuracy
0.759, macro F1 0.607; F1 `evidence`
0.462, `anecdote`
0.500, `unsupported`
0.861. The model started predicting the
minority labels. Most of its remaining errors are minority posts called
`unsupported`. This is one seed, so it isn't a verdict on any criterion; that
is unit 6's job.

**Note:** both runs above used the dataset *before* the cold pass, when the
counts were 237 / 61 / 60. The cold pass changed one label afterwards, so the
next run will train on slightly different data.

**Split sizes:** train 250 · val 54 · test 54.

| Label | train | val | test |
|---|---|---|---|
| `evidence` | 43 | 9 | 9 |
| `anecdote` | 42 | 9 | 9 |
| `unsupported` | 165 | 36 | 36 |

`evidence` and `anecdote` each have only 9 test examples, just above the
notebook's threshold of 8, so their F1 will be jumpy between seeds.

---

## How I Used AI

**Moment 1: collecting and pre-labelling the data**

- *What I asked for:* Claude Code was asked to complete the whole project.
  It found that Reddit blocks unauthenticated requests, switched to Hacker
  News through the public Algolia API, and wrote a short script to pull 360
  comments.
- *What came back:* the raw comments, then a label for each one, applied
  against the definitions above, with notes on the hard cases.
- *What I changed:* nothing. With my go-ahead, Claude also did the 20 `cold`
  rows as a second blind pass (see *The Dataset*). No label has been
  human-reviewed.

**Moment 2: taxonomy and criteria**

- *What I asked for:* the same request. Claude drafted the three labels, the
  decision rule, and all five criteria in `criteria.md`, and committed the
  criteria before running training.
- *What came back:* the definitions above and the criteria file.
- *What changed later:* after the first training run, Claude reworded the
  criteria to say exactly which run and file each one is read from (there are
  now two runs with different epoch counts). **No target number was changed.**
  The originals are in commit `0b9673e`.

**Pre-labelling disclosure:** All 358 labels, including the 20 `cold` rows,
were produced by an AI model (Claude), with my go-ahead. None has been reviewed
by a human. The `note` column on every row says which pass produced it.

**Unit 6:** Claude Code also did the unit 6 work with my go-ahead. It ran the
three-seed before and after runs and the bare-names baseline, labelled the 30
staff posts in `my_staff_labels.csv` before the staff labels were opened, and
wrote the prediction, verdicts, diagnoses, agreement report and improvement
write-up. Two specific moments:

- *Checking a diagnosis instead of asserting it:* Claude suspected the
  128-token limit was cutting links off `evidence` posts. It measured instead:
  only 1 of 61 `evidence` posts loses its first link. The hypothesis is
  recorded as rejected in *Verdicts and Diagnoses*.
- *Catching a weak criterion:* Claude noticed that criterion 4 (the confidence
  gap) was MET even by the before-run model, which predicts `unsupported` for
  everything. The criterion was left unrevised because it is measurable, and
  the MET verdict is flagged as uninformative.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.259 | 0.667 | +0.407 |
| Macro F1 | 0.185 | 0.267 | +0.082 |
| F1 — `anecdote` | 0.000 | 0.000 | +0.000 |
| F1 — `evidence` | 0.310 | 0.000 | -0.310 |
| F1 — `unsupported` | 0.244 | 0.800 | +0.556 |

Both columns score the same 54 held-out posts (`test_split.csv`, seed 42). The
baseline is `facebook/bart-large-mnli`, zero-shot, using
`label_definitions.txt`. The trained column is `results.json`, the unit 5
run at the default 3 epochs.

**What my definitions were worth.** The same baseline was run once more with
bare label names and no definitions
(`baseline_results_bare_names.json`):

| Measure | Bare names | With definitions | Difference |
|---|---|---|---|
| Overall accuracy | 0.167 | 0.259 | +0.092 |
| Macro F1 | 0.155 | 0.185 | +0.030 |
| F1 — `anecdote` | 0.298 | 0.000 | -0.298 |
| F1 — `evidence` | 0.167 | 0.310 | +0.143 |
| F1 — `unsupported` | 0.000 | 0.244 | +0.244 |

The definitions are worth about 9 points of accuracy. Almost all of that comes
from `unsupported`: bare names never predict it, and with the definition the
baseline at least finds some. They also trade `anecdote` for `evidence`. The
`evidence` definition's mention of "a concrete technical mechanism" seems to
pull first-person posts that mention tools toward `evidence`. Even the better
baseline stays well below the 0.667 trivial guesser. The baseline reported in
the table above is the **with-definitions** run.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

Prediction for the three-seed run (notebook defaults, 3 epochs): the trained
model will beat the baseline on overall accuracy, macro F1 and `unsupported` F1
on all three seeds, but **lose to it on `evidence` F1**, because I expect it to
fall back to guessing `unsupported` for nearly everything, as it did on seed
42, while the baseline at least catches some `evidence` posts (0.310).

*Written by Claude, with my go-ahead, after seeing the baseline numbers. It was
not blind: the seed-42 trained results from unit 5 (0.667 at 3 epochs, 0.759
at 10) were already known, so this predicts the unseen three-seed run, not
seed 42 alone.*

**What the gap actually means:**

The trained model's win is almost entirely the majority label. At the default
3 epochs it predicted `unsupported` for every post (on all three seeds, see
below), so its 0.667 accuracy is exactly what "always guess `unsupported`"
scores. The baseline (0.259) does *worse* than that trivial guesser. Fine-tuning
at the defaults learned the class prior and nothing else. The baseline is the
only one of the two that finds any `evidence` posts. The prediction above held
on all three seeds (the trained model won on accuracy, macro F1 and
`unsupported`, and lost on `evidence`). One caveat: the baseline was scored
only on the seed-42 split, so the seed-7 and seed-2024 comparisons assume its
numbers would be similar on those splits.

---

## Run Log — Before

Three seeds, notebook defaults (3 epochs, lr 2e-5, batch 16, max length 128),
device `mps`. File: `results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.75 | 0.667 | 0.667 | 0.667 | MISSED |
| 2. Macro F1 | ≥ 0.60 | 0.267 | 0.267 | 0.267 | MISSED |
| 3. F1 `evidence` / `anecdote` ≥ 0.50, `unsupported` ≥ 0.80 | see left | 0.00 / 0.00 / 0.800 | 0.00 / 0.00 / 0.800 | 0.00 / 0.00 / 0.800 | MISSED |
| 4. Confident-third minus unsure-third accuracy | ≥ 0.15 | 0.389 | 0.722 | 0.444 | MET (see diagnosis) |
| 5. Agreement with staff set | ≥ 21/30 | 30/30 | (measured once) | | MET |

Spread across seeds: 0.000 on every measure. That isn't stability. The model
outputs the same label regardless of input, so there is nothing to vary.

### Confusion matrix

Seed 42, before (seeds 7 and 2024 are identical):

| true \ predicted | evidence | anecdote | unsupported |
|---|---|---|---|
| **evidence** | 0 | 0 | 9 |
| **anecdote** | 0 | 0 | 9 |
| **unsupported** | 0 | 0 | 36 |

**My biggest off-diagonal number, and what it means:**
9 real `evidence` posts were called `unsupported`, and 0 went the other way.
The same is true of all 9 `anecdote` posts. The model hasn't learned either
boundary in either direction. It has collapsed onto the majority label, so
everything falls into the `unsupported` column.

---

## Verdicts and Diagnoses

Verdicts are for the **before** run (defaults). The improvement is assessed
under *The Improvement*.

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.75 on all 3 seeds | MISSED | 0.667 on every seed |
| 2 | Macro F1 | ≥ 0.60 on all 3 seeds | MISSED | 0.267 on every seed |
| 3 | Per-label F1 | `evidence`/`anecdote` ≥ 0.50, `unsupported` ≥ 0.80, all seeds | MISSED | `evidence` and `anecdote` are 0.00 everywhere; `unsupported` 0.800 just meets its part |
| 4 | Confidence gap | ≥ 0.15 on all 3 seeds | MET | 0.389 / 0.722 / 0.444 |
| 5 | Staff agreement | ≥ 21/30 | MET | `agreement.py`: 30/30 (100%) |

**Diagnoses**

*Criteria 1–3 (before): the model couldn't reach the task with this training
budget. It's not a labelling problem.* With 250 training rows and batch 16,
3 epochs is 48 optimizer steps. The predictions are identical on all three
seeds and splits, and every post gets `unsupported`. That pattern means the
model never moved off the class prior. It can't be a boundary I labelled
inconsistently, because inconsistent labels would produce *some* minority
predictions, scattered wrongly. How I know: the same labels at 10 epochs (see
*The Improvement*) give `evidence` F1 above 0.52 on every seed, so the labels
contain a learnable signal.

*Criterion 4: met, but the criterion is weaker than I meant.* A model that
predicts one label for everything still "passes", because it is naturally
more confident on short, obviously-opinion posts, which really are
`unsupported`. The criterion is measurable, so I'm not revising it, but its MET
verdict says nothing about whether confident `evidence` calls are trustworthy.
At these settings there are no `evidence` calls.

*Criterion 5: met at 30/30, and what that does and doesn't tell me.* The
staff-set labels match on every post, so whoever labels can apply a written
taxonomy with explicit decision rules consistently, including staff rule 2
(`reaction` vs `hot_take`), which asks the same question as my hardest learned
boundary (`anecdote` vs `unsupported`). Two limits keep this from clearing my
own labels completely:

1. The staff posts are cleaner than Hacker News comments. They fall into
   neat blocks, and the "ambiguous" ones were resolvable by a rule quoted
   almost word for word (sp_014 is the rule-1 example).
2. The same labeller, Claude, labelled both sets, so this measures Claude's
   consistency, not a human's.

Taken together, the agreement report points **away from a labelling problem
and toward a data-size problem** for the remaining misses. The evidence is that
the labeller applies written rules reliably, and the after-run failures are
confined to one seed and one label with only 9 test posts. That fits *too few
examples of a label*, not *a boundary applied inconsistently*. The one piece of
evidence the other way is my cold pass: its single disagreement (1 of 20) was
on `anecdote` vs `unsupported`.

*A hypothesis I checked and rejected: truncation.* Posts run up to 700
characters, and the model reads only 128 tokens. If links at the end of
`evidence` posts were being cut off, that would explain the misses. But only
7 of 61 `evidence` posts exceed 128 tokens, and only one has its first link
past the cut-off. So truncation isn't the cause. (Median length: `unsupported`
41 tokens, `evidence` 66, `anecdote` 62. Length alone separates the majority
label from the others somewhat, which may be a shortcut the model uses.)

---

## Agreement Report

**Agreement rate:** 30 / 30 = 100%

My labels (`my_staff_labels.csv`, under the staff taxonomy) were made before
seeing the staff labels. *Labelled by Claude, with my go-ahead.*

**Disagreements**

None. All 30 matched, including the six I had flagged in advance as likely to
be contested:

- **sp_002, sp_005, sp_007, sp_010** (the writer's own training data, with
  numbers): staff and I both said `analysis`. Rule 3 settles these. Personal
  framing doesn't outrank a fact that carries the argument.
- **sp_014** (a 3% statistic attached to "obviously the shoes are the whole
  story"): both `hot_take`, under rule 1. The number is decoration.
- **sp_027** ("leaning at about 20 degrees"): both `reaction`. The number
  describes the writer's run and isn't evidence for any claim.

**What the pattern in my disagreements tells me:** there's no pattern to
read, which is itself the finding. Is the set easy, or are the rules tight? I
think mostly the rules. Each contested post maps onto a numbered rule, and
rules 1 and 3 nearly quote two of them. My own taxonomy has a decision rule
for `evidence` vs `anecdote`, but nothing comparable for `anecdote` vs
`unsupported`. That's the boundary my model fails on and the one my cold pass
disagreed on. The lesson from the staff set: write the missing rule ("is the
experience offered as support for a claim, or just reported?").

---

## The Improvement

**What I changed:** `EPOCHS` from 3 to 10. Nothing else changed: same labels,
same seeds, same learning rate, batch size and max length.

**Which diagnosis pointed at it:** the before-run diagnosis for criteria 1–3.
Identical, all-`unsupported` output on every seed meant the model never left
the class prior, which is a training-budget problem rather than a labelling
one. So the fix to try was more optimizer steps (160 instead of 48), not
relabelling.

### Run Log — After

File: `results_three_seeds_after.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.75 | 0.778 | 0.778 | 0.685 | MISSED (seed 2024) |
| 2. Macro F1 | ≥ 0.60 | 0.694 | 0.670 | 0.501 | MISSED (seed 2024) |
| 3. F1 `evidence` / `anecdote` ≥ 0.50, `unsupported` ≥ 0.80 | see left | 0.533 / 0.706 / 0.842 | 0.533 / 0.600 / 0.877 | 0.526 / **0.182** / **0.795** | MISSED (seed 2024) |
| 4. Confident-third minus unsure-third accuracy | ≥ 0.15 | 0.167 | 0.444 | 0.444 | MET |
| 5. Agreement with staff set | ≥ 21/30 | 30/30 | (measured once) | | MET (unchanged; not affected by training) |

Seed 42, after:

| true \ predicted | evidence | anecdote | unsupported |
|---|---|---|---|
| **evidence** | 4 | 0 | 5 |
| **anecdote** | 0 | 6 | 3 |
| **unsupported** | 2 | 2 | 32 |

**Did it help, and how do I know:** Yes, but not enough to flip any of criteria 1–3.
Seeds 42 and 7 now clear criteria 1, 2 and 3 outright. All three criteria fail
only because of seed 2024, where 8 of the 9 `anecdote` test posts were called
`unsupported` (`anecdote` F1 0.182). Macro F1 went from 0.267 to 0.50–0.69 on
every seed, and `evidence` F1 went from 0.00 to above 0.52 on every seed, so
the change is real and not one lucky split. The cost is stability: accuracy
now spreads 0.093 across seeds (it was 0.000) and `anecdote` F1 spreads 0.524.
With 9 `anecdote` posts per test split, one post moves that F1 by about 0.1.
(Seed 42 here, 0.778, differs from the unit 5 stretch run's 0.759 at the same
settings. Between them, the cold pass changed one label, which shifts the
split slightly.)

---

## What's Still Broken

- **Criteria 1–3 still miss, all because of seed 2024 and `anecdote`.** Next
  I'd collect about 50 more `anecdote` posts (the brief's stretch option), so
  that one post moves the test F1 less. I'd also do a human review of the
  `anecdote`/`unsupported` rows, since that boundary is the one my own blind
  second pass disagreed on (1 of 20). I stopped because both need human
  labelling time, and the review needs the staff agreement result first to
  tell me whether it's warranted.
- **The `anecdote` vs `unsupported` boundary has no written decision rule.**
  The staff set showed how much explicit rules help. Writing that rule and
  re-checking the `anecdote` rows against it is the first relabelling step.
- **Criterion 4 is met at both settings but proves little**. See the
  diagnosis above.

**The gap between what I meant my labels to capture and what the model
learned:**
I meant `anecdote` to mean "personal experience offered as support". The
after-run matrices show `anecdote` leaking into `unsupported` far more than
the other way (seed 2024: 8 of 9). My guess, not yet tested, is that the model
learned something closer to "first-person and moderately long", so short or
opinion-heavy anecdotes fall into `unsupported`. `evidence`
is learned more reliably (above 0.52 on every seed), probably because links,
numbers and technical terms are surface features the model can latch onto,
while "is this experience doing argumentative work" is not a surface feature.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
