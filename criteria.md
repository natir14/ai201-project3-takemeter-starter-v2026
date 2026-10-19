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

All five targets are measured on the held-out test split and must hold on
**every one of the three seeds (42, 7 and 2024)**. A miss on any one seed is a
miss.

My dataset: 209 r/nba comments: 144 `reaction` (68.9%), 44 `hot_take`
(21.1%), 21 `analysis` (10.0%). The split is stratified 70/15/15, so each test
split holds about 32 comments: roughly 22 `reaction`, 7 `hot_take` and 3
`analysis`.

---

## 1. Overall accuracy is at least 0.75 on the held-out set, on all three seeds.

**Why this target:**

69% of my comments are `reaction`, so a model that answers `reaction` every
time already scores about 0.69. 0.75 asks the model to beat that do-nothing
strategy by about two more correct comments out of 32 on every seed. Anything
lower would let a model that never learned `hot_take` or `analysis` pass.

---

## 2. Macro F1 is at least 0.60 on the held-out set, on all three seeds.

**Why this target:**

Accuracy can be inflated by my big `reaction` class; macro F1 weighs all three
labels equally. The always-`reaction` model gets a macro F1 of about 0.27
(0.82 on `reaction`, 0 on the other two). 0.60 means the model has to be
genuinely useful on at least one of my small labels, not just the big one.

---

## 3. Confidence-based prediction accuracy

The model must achieve at least **80% accuracy on the most-confident third of
predictions** on the held-out test set for each of the three seeds (42, 7, and
2024).

**Why this target:**

My dataset contains 209 NBA Reddit comments, with 144 reactions, 44 hot takes,
and 21 analysis posts. Some comments are difficult to classify because the
boundaries overlap. I expect the model to be more reliable when it is
confident. I chose 80% because confident predictions should be more accurate
than simply predicting the majority label, which represents 68.9% of my
dataset.

---

## 4. `hot_take` F1 is at least 0.50 on the held-out set, on all three seeds.

**Why this target:**

`hot_take` has 44 examples, only about 7 per test split, so one or two
mistakes move its F1 a lot. It is also on both of my hard boundaries: against
`analysis` (a claim with thin support) and against `reaction` (a joking
prediction like "Just give him the MVP now"). 0.50 is lower than my overall
performance targets because `hot_take` has fewer examples and overlaps with
both `reaction` and `analysis`. It is still well above the 0 that a majority
guesser gets.

---

## 5. `analysis` F1 is at least 0.40 on the held-out set, on all three seeds.

**Why this target:**

`analysis` is my smallest label: 21 examples, about 3 in each test split. On 3
examples, F1 is very jumpy. Getting just 1 of 3 right with 1 false alarm
already gives 0.40, and missing all 3 gives 0. I set it the loosest of my five
because of that, but above 0 so the model has to find at least some real
reasoning posts on every seed.

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
