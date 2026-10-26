# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

> **How every criterion is measured.** Each one uses the unit 6 three-seed run
> (notebook section 6, seeds 42, 7 and 2024) on `labels.csv`, scored on each
> seed's held-out test split, with whatever training settings are in the
> notebook at that time. A criterion is **met** only if it holds on all three
> seeds. Criterion 5 is the exception: it is measured once, with `agreement.py`.
>
> *Wording revised after the first training run to remove this ambiguity. Every
> target number is unchanged from the original commit (`0b9673e`).*

## 1. Overall accuracy of at least 0.75 on every seed

Read from `accuracy` for each seed in `results_three_seeds_before.json`.

**Why this target:** `unsupported` is about 66% of my 358 rows, so always
guessing it already scores about 0.66. 0.75 forces the model to get a real
share of `evidence` and `anecdote` right. Anything under that has barely
beaten the majority guess.

---

## 2. Macro F1 of at least 0.60 on every seed

Read from `f1_macro` for each seed.

**Why this target:** accuracy hides the two thin labels and macro F1 doesn't.
Always guessing `unsupported` scores a macro F1 of about 0.27 on this data.
0.60 is what I'd call useful with about 60 examples per minority label. It
doesn't require the model to be good at everything.

---

## 3. F1 of at least 0.50 for `evidence` and for `anecdote`, and at least 0.80 for `unsupported`, on every seed

Read from `per_label.<label>.f1` for each seed. All three label conditions must
hold on all three seeds.

**Why this target:** `evidence` (61 rows) and `anecdote` (61 rows) each get only
about 9 posts per test split, so one post moves their F1 a lot, which is why
their floor is lower. `unsupported` has about 36 test posts and gets the higher
bar. 0.50 on a thin label still means the model catches at least half of them
without flooding that label with false positives.

---

## 4. The most-confident third is at least 0.15 more accurate than the least-confident third, on every seed

Read as `confidence.accuracy_most_confident_third` minus
`confidence.accuracy_least_confident_third` for each seed.

**Why this target:** if this classifier sorted a feed, a confident `evidence`
call should be trustworthy in a way a hesitant one isn't. Each third holds
about 18 posts, so one post moves a third's accuracy by about 0.055. A gap of
0.15 is roughly three posts, which is more than one or two lucky calls can
produce.

---

## 5. At least 21 of 30 staff posts (70%) labelled the same as staff

Read as the agreement rate `agreement.py` prints, comparing
`my_staff_labels.csv` against `data/staff_labels.csv`.

**Why this target:** this one measures the labeller, not the model. My hardest
boundary (a personal story that also includes a link or a number) needed a
written rule to settle. If someone else's written rules can't be applied 70% of
the time, mine probably weren't applied consistently either. 70% is well above
the roughly 33% from guessing among three labels, and leaves room for the posts
staff picked because they're ambiguous.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
