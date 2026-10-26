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

TakeMeter sorts comments from **r/nba**, the main public NBA discussion
community on Reddit, into three labels: `analysis` (basketball reasoning
backed by real evidence or explanation), `hot_take` (a strong claim or
prediction without enough support) and `reaction` (emotion, jokes, sarcasm or
surprise). The goal is to separate the comments that argue something from the
ones that just assert or react. It is a fine-tuned `distilbert-base-uncased`
classifier trained on 209 comments I collected and labelled.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

The definitions are also in [label_definitions.txt](label_definitions.txt),
which `baseline.py` reads.

### `analysis`

**Definition:** Basketball reasoning supported by meaningful evidence or
explanation, such as a stat, a specific play or matchup, a tactical or rules
mechanism, or a concrete comparison that explains why the claim holds.

**Example 1:**
> Injury risk.
>
> If a defender anticipates this shot, they can establish position and box him
> out while he’s watching the ball. That defender can then jump with as much
> force as they want. Wemby would get called with a displacement foul or over
> the back.
>
> Welby could easily fall awkwardly or receive a headbutt to his chin

**Example 2:**
> Something underrated on this clip is he bites on the fake then the dude goes
> up as he is in the air and still gets blocked, its why he can afford to bite
> on alot of fakes.

### `hot_take`

**Definition:** A strong basketball claim or prediction (best ever, GOAT,
overrated, will win MVP) offered with insufficient reasoning, where any
support is vague, generic, or just restates the claim.

**Example 1:**
> Ive seen enough this kid is gonna be a GOAT

**Example 2:**
> Hate to sound hyperbolic, but I think he’s clearly the best passer of the
> 21st century

### `reaction`

**Definition:** An emotional response, joke, sarcasm, excitement, or surprise
about what just happened, which makes no real basketball argument.

**Example 1:**
> He makes Sabonis look like a baby

**Example 2:**
> Why doesn’t Wemby just eat the opposing team

### The hardest boundary

**Which two labels:** `hot_take` and `reaction`. r/nba constantly makes real
claims in a joking voice ("Just give him the MVP now 😂"), so the same comment
reads as either hype or a prediction.

**The decision rule I used every time:** Apply the labels in priority order and
stop at the first that fits:

1. **Meaningful reasoning wins.** If the comment explains *why* with something
   a reader could check or follow, it is `analysis`, even if the tone is
   heated or joking. Test: delete the reason. If the comment lost an argument,
   it's `analysis`; if it only lost decoration, it isn't.
2. **Otherwise, a strong unsupported claim is `hot_take`**, even when it's
   phrased as a joke. A concrete prediction or ranking (an award, "GOAT",
   "best passer") counts as a claim no matter the tone.
3. **Otherwise it's `reaction`.** That covers feelings, humor, sarcasm, hype
   and disbelief, including a checkable fact used only to express amazement
   ("4 dribbles!").

**Exclusion rule:** comments whose meaning depends on the parent comment or the
clip ("Was also my thought.", "Preseason too") were left out of the dataset,
because the model only ever sees the text column.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public r/nba comment sections, copied by hand
as whole comments. 181 comments come from the thread for a Wembanyama clip in
which he blocks a shot and then euro-steps into a self-lob off the glass,
against the Kings. 28 come from a thread on a Nikola Jokić cross-court pass. I
left out ads, duplicates, comments cut off mid-text, one mostly-French comment,
and 7 comments that only made sense with their parent comment or the clip.

**How I labelled them:**
- **20 cold rows** (`note` = `cold`): I labelled the first 20 comments myself,
  with only my definitions and no tools.
- **189 pre-labelled rows** (`note` starts with `prelabeled`): labelled by AI.
  Claude checked them against the decision rule and corrected 11 for
  consistency with earlier decisions; each of those notes says
  `relabeled from <old label>` and why. 35 rows have notes marked
  `hard case`.

All details are in How I Used AI below.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reaction` | 144 | 68.9% |
| `hot_take` | 44 | 21.1% |
| `analysis` | 21 | 10.0% |
| **Total** | **209** | 100% |

`reaction` sits just under the 70% cap. Highlight threads produce mostly
reactions, and the Jokić thread was added partly to bring in more `analysis`
and `hot_take`.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Just give him the MVP now we all know it's coming 😂"
>
> *Could have been:* `hot_take` or `reaction`
>
> *I chose, because:* `hot_take`. The laughing emoji makes it read like hype,
> but it makes a concrete award prediction with no support, and rule 2 puts a
> claim ahead of tone. The same call was applied to "I've seen enough. Give
> him the MVP right now", which had first come in labelled `reaction`.

**2.**
> *The post:* "He crossed the court in 4 dribbles, 4 dribbles!"
>
> *Could have been:* `analysis` or `reaction`
>
> *I chose, because:* `reaction`. "4 dribbles" is a checkable fact, but it's
> used to express amazement, not to support an argument. Rule 1 needs
> reasoning, not just a fact. Several near-copies ("4 fuckin dribbles…. 4!")
> got the same label.

**3.**
> *The post:* "I agree and would go a step further and say he firmly in
> conversation for the passing GOAT. I didn’t get to see Magic live but it’s
> hard for me to think of someone who has better highs, more highs and less
> lows."
>
> *Could have been:* `analysis` or `hot_take`
>
> *I chose, because:* `hot_take`. It came in pre-labelled `analysis` because it
> sounds reasoned, but it fails the deletion test: "better highs, more highs
> and less lows" is vague support for a GOAT claim, not evidence.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Trained on **cpu** (torch 2.14.1+cpu) in about 2 minutes 13
seconds.

**Anything I changed from the defaults, and why:** Nothing. Every setting is
the notebook default.

**How it was run:** I ran the code cells for sections 1–5 of `takemeter.ipynb`
from the command line with this project's `.venv` Python, not inside VS Code.
So **the outputs shown in the notebook are not current**: section 1 still says
`labels.csv` wasn't found, and sections 2–5 show no output. The recorded
results are in [results.json](results.json) and
[test_split.csv](test_split.csv). I checked that test_split.csv is exactly
the seed-42 split of labels.csv.

**Split sizes:** train 145 · validation 32 · test 32 (stratified 70/15/15)

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 15 | 3 | **3** |
| `hot_take` | 30 | 7 | **7** |
| `reaction` | 100 | 22 | 22 |

`analysis` (3) and `hot_take` (7) each have **fewer than 8 test examples**, so
their F1 scores will swing a lot between seeds.

**Results (seed 42, from results.json):**

| Measure | Value |
|---|---|
| Accuracy | 0.688 |
| Macro F1 | 0.272 |
| F1 — `reaction` | 0.815 |
| F1 — `hot_take` | 0.000 |
| F1 — `analysis` | 0.000 |
| Accuracy, most-confident third (10 posts) | 1.000 |
| Accuracy, least-confident third (10 posts) | 0.200 |

**The model predicted `reaction` for all 32 test comments.** 0.688 accuracy
is just the share of `reaction` in the test split (22 of 32), and 0.272 macro
F1 is what always answering `reaction` scores. With 145 training comments, 3
epochs and batch size 16, the model made only about 27 weight updates, and the
final training loss (0.71) was only a little below what predicting the label
proportions gives. The full evaluation against my criteria is next unit's
work.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* For Claude to merge a 111-row file into labels.csv. It
  held 97 real comments from the Wembanyama thread with provisional labels,
  plus 14 AI-written `analysis` examples.
- *What came back:* Claude added 92 of the real comments and left out 5 that
  needed the clip or a parent comment, or were mostly in French. It changed 4
  labels to match earlier decisions, for example "I've seen enough. Give him
  the MVP right now" from `reaction` to `hot_take`. It held back all 14
  synthetic rows: they weren't real r/nba comments, their polished coaching
  language would teach the model "formal writing = `analysis`", and some would
  land in the test split and inflate the scores.
- *What I changed:* I kept the synthetic rows out, so every row in labels.csv
  is a real r/nba comment, and I kept the 4 relabels. Instead, I collected
  more real comments from a Jokić passing thread to fill out the thin labels.

**Moment 2**

- *What I asked for:* For Claude to check my five draft acceptance criteria,
  which were accuracy, macro F1, and F1 for each of the three labels.
- *What came back:* Claude pointed out that the five covered only two of the
  template's areas (overall accuracy and per-label performance), while the
  template asks for at least three.
- *What I changed:* I replaced criterion 3 (`reaction` F1 ≥ 0.75) with a
  confidence criterion: at least 0.80 accuracy on the most-confident third of
  predictions. I had Claude confirm that the notebook records this measure in
  results.json before I committed it.

**Pre-labelling disclosure:** 189 of the 209 rows are AI-pre-labelled and
marked `prelabeled` in the `note` column. The 20 rows marked `cold` I labelled
myself with no tools. Here is how the 189 rows were labelled:

| Batch | Rows | How the labels were made |
|---|---|---|
| First 19 comments from the Wembanyama thread | 19 | Claude labelled them from my definitions. |
| Next batch from the same thread | 50 | Claude labelled them from my definitions. |
| Imported file of Wembanyama-thread comments | 92 | They arrived with provisional labels from an earlier AI-assisted pass. Claude checked them and changed 4. |
| Jokić thread | 28 | They arrived pre-labelled. Claude checked them and changed 7. |

**What was and wasn't checked by a human:**
- The 20 cold rows are the only rows I labelled entirely on my own.
- The 11 corrections were made by Claude, applying my decision rule. I
  accepted them, and each is recorded in the `note` column with the original
  label and the reason.
- I have not yet confirmed a row-by-row review of all 189 pre-labelled rows,
  so treat them as AI labels checked by AI for consistency, not as
  independently verified human labels.

**Other AI help:**
- Claude turned my specifications for the three labels, the priority rule and
  the exclusion rule into the wording in label_definitions.txt.
- I chose all five targets in criteria.md. Claude drafted the reasons under
  criteria 1, 2, 4 and 5 from my dataset counts; I wrote the reason under
  criterion 3.
- Claude ran the training code and committed the files at my request.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.219 | 0.688 | +0.469 |
| Macro F1 | 0.222 | 0.272 | +0.049 |
| F1 — `analysis` | 0.333 | 0.000 | -0.333 |
| F1 — `hot_take` | 0.333 | 0.000 | -0.333 |
| F1 — `reaction` | 0.000 | 0.815 | +0.815 |

Baseline: zero-shot `facebook/bart-large-mnli` using my label definitions,
scored on the same 32 held-out comments as the trained model (seed 42).
Full results are in `baseline_results.json`.

**What I predicted before I looked:**
I predicted that the baseline might achieve a higher macro F1 than the trained
model, because it could recognize the minority labels (`analysis`, `hot_take`)
that the trained model missed.

This was not a blind prediction. The trained model's scores (accuracy 0.688,
macro F1 0.272, F1 of 0.000 on `analysis` and `hot_take`) were already recorded
in Unit 5 when I wrote it, so I already knew the trained model had no minority
label predictions to defend.

The prediction was not supported. Baseline macro F1 was 0.222, compared with
0.272 for the trained model.

**What the gap actually means:**
The trained model has higher overall accuracy (+0.469), but it predicted
`reaction` for all 32 held-out comments. Its 0.688 accuracy is the share of
`reaction` in the test set (22 of 32), so it learned the majority class and
nothing else. It cannot tell an `analysis` or `hot_take` comment apart from a
`reaction`.

The baseline does the opposite. It found every `analysis` comment and 4 of 7
`hot_take` comments, but it never predicted `reaction` correctly (F1 0.000 on
22 comments). It labeled all 22 of those quick reactions as `hot_take` (13)
or `analysis` (9).

The trained model's macro F1 lead (+0.049) is small, and it comes entirely from
`reaction`, the one label it predicts. Neither model separates the three labels
in a useful way. Fine-tuning taught the model the label distribution, not the
difference between the labels.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.75 | 0.688 | 0.688 | 0.688 | MISSED |
| 2. Macro F1 | ≥ 0.60 | 0.272 | 0.272 | 0.272 | MISSED |
| 3. Accuracy on most-confident third | ≥ 0.80 | 1.000 | 1.000 | 0.900 | MET |
| 4. F1 — `hot_take` | ≥ 0.50 | 0.000 | 0.000 | 0.000 | MISSED |
| 5. F1 — `analysis` | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED |

Numbers are from `results_three_seeds_before.json`. Each seed was tested on 32
held-out comments: 22 `reaction`, 7 `hot_take` and 3 `analysis`. Training ran
on **CPU** (torch 2.14.1+cpu), with the unchanged Unit 5 settings: 3 epochs,
learning rate 2e-5, batch size 16, max length 128. For reference, accuracy on
the least-confident third was 0.200 / 0.400 / 0.600. Each third is 10
comments.

**All three seeds predicted `reaction` for all 32 test comments.** The scores
are identical across seeds because each stratified split has the same 22 / 7 /
3 label counts. A model that always answers `reaction` gets exactly 22 of 32
right (0.688) and an F1 of 0 on both minority labels.

**Zero spread does not show the model generalizes reliably.** A spread of
0.000 normally means a stable measurement. Here it only means the model gives
the same answer for every comment, whatever split it trained on. It always
predicts the majority class, so its scores depend on the label counts and not
on what it learned. Validation accuracy also stayed at 0.6875 after every
epoch on every seed, so the model never predicted a minority label during
training either.

Criterion 3 is met, but only in a narrow sense. Every prediction is
`reaction`, so the most-confident third is just the 10 comments the model
rated most likely to be `reaction`, and those mostly are. The model does rank
comments by how likely they are to be `reaction`, but that ranking never
changed its answer on any comment.

**Possible explanations, not yet proven causes:**
- **Class imbalance.** `reaction` is 69% of the data (144 of 209), so always
  guessing it is a strong strategy for the training loss.
- **Insufficient training.** Each run was only about 30 optimizer steps on 145
  training comments. Training loss was still falling at the end of epoch 3
  (about 0.71 to 0.80).

Neither of these has been tested yet.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42. Seeds 7 and 2024 produced the identical matrix.

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 0 | 0 | 3 |
| **hot_take** | 0 | 0 | 7 |
| **reaction** | 0 | 0 | 22 |

**My biggest off-diagonal number, and what it means:**
7 real `hot_take` comments were predicted as `reaction`, and 0 went the other
way. This is not a confusion between two similar labels. The model never
predicts anything except `reaction`, so every `hot_take` and every `analysis`
comment ends up in the `reaction` column. `hot_take` → `reaction` is the
largest cell only because `hot_take` is the bigger minority label (7 test
comments against 3 for `analysis`). The model learned the label distribution,
not the boundary between `hot_take` and `reaction`.



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

**Agreement rate:** 29 / 30 = 96.7%

**How these labels were made — read this before the rate.** I labelled posts
1–5 independently. For posts 6–30 I accepted labels suggested by ChatGPT. The
provenance of every label is in `my_staff_labels_source.csv`. So 96.7% is
**not** a measure of independent human agreement:

| Source | Agreed with staff |
|---|---|
| My own labels (posts 1–5) | 4 / 5 (80%) |
| ChatGPT-suggested, accepted by me (posts 6–30) | 25 / 25 (100%) |

Six of the seven posts staff flagged as hard fall in the ChatGPT-suggested
set. The only evidence here about how *I* apply a written rule is five posts,
which is too few to say much.

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
> *The post:* "Everyone blames the hill at 18 miles. I split the field into
> people who ran the first half under 1:45 and over, and the slowdown after 18
> was almost identical for both groups. If the hill were the problem the fast
> starters would fall apart harder."
>
> *Staff said / I said:* `analysis` / `hot_take` (my own label, not
> AI-suggested)
>
> *My call, and why:* Staff is right. Rule 1 asks whether the checkable fact
> carries the argument, and the definition of `analysis` lists a named
> comparison as such a fact. Here the comparison (sub-1:45 vs. slower first
> halves, slowdown after mile 18) *is* the argument. Without it, "the hill
> isn't the problem" has nothing behind it. I think the combative opening
> ("Everyone blames…") pulled me toward `hot_take`. The closest case for my
> label is that the result is only "almost identical", with no number, which
> is the reason staff gave for calling sp_020 a hot take. But sp_020 offers no
> method at all, and this post names a specific threshold and a comparison
> anyone could rerun.
>
> *Which pile:* (a). The rule covered it and I applied it loosely. I reacted
> to the tone instead of checking whether the fact was load-bearing.

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
