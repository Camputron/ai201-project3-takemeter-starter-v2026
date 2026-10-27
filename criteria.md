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

## 1. Overall accuracy is at least 0.75 on the held-out test split, on all three seeds

**Why this target:** `unsupported` is about 66% of my 358 rows, so a model that
always guesses `unsupported` already scores about 0.66. 0.75 means the model
has to get a real share of the `evidence` and `anecdote` posts right as well.
Anything below that hasn't learned much beyond the majority label.

> **Revised in unit 6:** Overall accuracy of at least 0.75 on every seed, read from `accuracy` in `results_three_seeds_<run>.json`. Measured on the three-seed run (notebook section 6, seeds 42, 7, 2024), on each seed's held-out test split, with whatever training settings the notebook had at the time. MET only if it holds on all three seeds.
>
> **Why revised:** The original never said which run it is judged on. By unit 6 there were two training configurations (3 and 10 epochs) and four results files, so it couldn't produce a single verdict. **The target is unchanged.** *(Reworded by Claude, with my go-ahead, after the first training run; moved under the original in unit 6.)*

---

## 2. Macro F1 is at least 0.60 on the held-out test split, on all three seeds

**Why this target:** accuracy hides the two thin labels, and macro F1 doesn't,
because it weights all three labels equally. The always-`unsupported` guesser
scores a macro F1 of about 0.27 here. 0.60 is a level I'd call useful with
about 60 examples per minority label. It doesn't require the model to be good
at every label.

> **Revised in unit 6:** Macro F1 of at least 0.60 on every seed, read from `f1_macro`. Measured on the three-seed run (notebook section 6, seeds 42, 7, 2024), on each seed's held-out test split, with whatever training settings the notebook had at the time. MET only if it holds on all three seeds.
>
> **Why revised:** The original never said which run it is judged on. By unit 6 there were two training configurations (3 and 10 epochs) and four results files, so it couldn't produce a single verdict. **The target is unchanged.** *(Reworded by Claude, with my go-ahead, after the first training run; moved under the original in unit 6.)*

---

## 3. Every label has an F1 of at least 0.50, and `unsupported` has an F1 of at least 0.80, on all three seeds

**Why this target:** `evidence` (61 rows) and `anecdote` (60 rows) each get only
about 9 posts in a 15% test split, so their F1 will jump around between seeds.
That's why the floor for them is lower. `unsupported` has 237 rows, about 36 of
them in test, so I hold it to a higher bar. 0.50 on a thin label still means
the model finds at least half of them without flooding that label with false
positives.

> **Revised in unit 6:** F1 of at least 0.50 for `evidence` and for `anecdote`, and at least 0.80 for `unsupported`, read from `per_label.<label>.f1`. All three conditions on all three seeds. Measured on the three-seed run (notebook section 6, seeds 42, 7, 2024), on each seed's held-out test split, with whatever training settings the notebook had at the time. MET only if it holds on all three seeds.
>
> **Why revised:** The original never said which run it is judged on. By unit 6 there were two training configurations (3 and 10 epochs) and four results files, so it couldn't produce a single verdict. **The target is unchanged.** *(Reworded by Claude, with my go-ahead, after the first training run; moved under the original in unit 6.)*

---

## 4. The most-confident third of test predictions is at least 0.15 more accurate than the least-confident third, on all three seeds

(Measured with `confidence.accuracy_most_confident_third` minus
`confidence.accuracy_least_confident_third` in `results.json`.)

**Why this target:** if this classifier were used to sort a feed, a confident
`evidence` call should be one you can trust more than a hesitant one. With
about 54 test posts, each third has about 18 posts, so a single post moves a
third's accuracy by about 0.055. 0.15 is a gap of roughly three posts, which
is larger than what one or two lucky calls would produce.

> **Revised in unit 6:** `confidence.accuracy_most_confident_third` minus `confidence.accuracy_least_confident_third` is at least 0.15 on every seed. Measured on the three-seed run (notebook section 6, seeds 42, 7, 2024), on each seed's held-out test split, with whatever training settings the notebook had at the time. MET only if it holds on all three seeds.
>
> **Why revised:** The original never said which run it is judged on. By unit 6 there were two training configurations (3 and 10 epochs) and four results files, so it couldn't produce a single verdict. **The target is unchanged.** *(Reworded by Claude, with my go-ahead, after the first training run; moved under the original in unit 6.)*

---

## 5. My labels agree with the staff labels on at least 21 of the 30 staff posts (70%), using `agreement.py`

**Why this target:** this criterion is about the labeller (me), not the model.
My hardest boundary, a personal story that also includes a link or a number,
took a written rule to settle. If I can't apply someone else's written rules
70% of the time, I probably applied mine inconsistently too. 70% sits clearly
above the roughly 33% you'd get by guessing among three labels. It leaves room
for the deliberately ambiguous posts staff included.

> **Revised in unit 6:** At least 21 of 30 staff posts (70%) labelled the same as staff, read as the rate `agreement.py` prints when comparing `my_staff_labels.csv` against `data/staff_labels.csv`. Measured once, not per seed.
>
> **Why revised:** The original never said which run it is judged on. By unit 6 there were two training configurations (3 and 10 epochs) and four results files, so it couldn't produce a single verdict. **The target is unchanged.** *(Reworded by Claude, with my go-ahead, after the first training run; moved under the original in unit 6.)*

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
