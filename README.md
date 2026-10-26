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

**How I labelled them:** **All 358 rows were pre-labelled by an AI (Claude,
via Claude Code)** applying the definitions and decision rule above. Every row's
`note` column says so. **No rows are marked `cold`. The 20 unaided cold labels
the brief asks for have not been done yet**, and no row has yet been
re-read and corrected by a human. See *How I Used AI*.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `unsupported` | 237 | 66.2% |
| `evidence` | 61 | 17.0% |
| `anecdote` | 60 | 16.8% |
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
- *What I changed:* nothing yet. **These labels still need a human pass.**

**Moment 2: taxonomy and criteria**

- *What I asked for:* the same request. Claude drafted the three labels, the
  decision rule, and all five criteria in `criteria.md`, and committed the
  criteria before running training.
- *What came back:* the definitions above and the criteria file.
- *What I changed:* nothing yet.

**Pre-labelling disclosure:** All 358 labels were produced by an AI model
(Claude) and none has been human-reviewed yet. Every row's `note` column
says `pre-labelled by Claude (AI); needs human review`. There are no `cold`
rows.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
