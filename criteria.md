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

## 1. Overall accuracy on the held-out test split is at least 0.60.

**How to check:** read `accuracy` in `results.json` for each of the three seeds. All three must be 0.60 or higher.

**Why this target:** 0.60 is realistic for a model trained on only 150–200 labeled comments.

---

## 2. Every label has an F1 of at least 0.50 on the held-out test split.

**How to check:** read each label's `f1` under `per_label` in `results.json` for each of the three seeds. `analysis`, `hot_take` and `experience` must all be 0.50 or higher.

**Why this target:** Every label should do somewhat better than guessing, but I'm not holding the blurry pair (`analysis` vs `hot_take`) to a hard standard. 0.50 is a sweet spot between the weak labels and the blurry pair.

---

## 3. No label is more than 60% of the rows in labels.csv.

**How to check:** count the rows for each label in `labels.csv` and divide by the total. The largest share must be 60% or lower.

**Why this target:** From reading Reddit, most comments are hot takes with no evidence or personal experiences, so I expect the data to skew toward those. 60% allows for that skew without letting any one label dominate.

---

## 4. My labels match the staff labels on at least 70% of the same posts.

**How to check:** in next unit's agreement report, compare my label with the staff label for each shared post. The percent that match must be 70% or higher.

**Why this target:** Most posts should get the same label from me and from staff, but the hard boundary (`analysis` vs `hot_take`) can still split, so I'm not asking for near-perfect agreement.

---

## 5. On the held-out test split, accuracy on the model's most-confident third of predictions is at least 0.75.

**How to check:** read `accuracy_most_confident_third` under `confidence` in `results.json` for each of the three seeds. All three must be 0.75 or higher.

**Why this target:** I expect the model to be right most of the time when it is confident. With about 25 test posts, a third is about 8 predictions, so 0.75 allows at most 2 wrong.

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
