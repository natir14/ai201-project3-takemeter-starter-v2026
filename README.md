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
