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

TakeMeter sorts public Reddit comments about healthy eating, nutrition, protein and sugar into three kinds, based on how the writer supports what they say: `analysis` (a claim with a named, findable source), `hot_take` (a confident claim with nothing checkable behind it) and `experience` (what happened to the writer personally). I fine-tuned a small model (`distilbert-base-uncased`) on 189 comments I collected and labelled, so it can tell a sourced argument from an unsupported opinion from a personal story. The community is "people who talk about healthy eating on Reddit", and the labels came from what I saw in the comments I read, not from a textbook.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes a claim backed by a named, findable source, such as a link, or a named study, paper, book or statistic.

**Example 1:**
> Cutting out sugar will make you stop liking sweets.
>
> https://www.sciencedaily.com/releases/2026/03/260318033107.htm

**Example 2:**
> organic doesn't mean healthy.
>
> "all natural" doesn't mean anything
>
> there are like 60+ names for sugar :P
>
> https://www.healthline.com/nutrition/56-different-names-for-sugar

### `hot_take`

**Definition:** A confident claim about food or health with no named, findable source behind it.

**Example 1:**
> Fruit is not equal to candy.

**Example 2:**
> Saturated fat doesn’t cause heart disease.

### `experience`

**Definition:** Describes what happened to the writer personally, such as their results, symptoms or routine, without claiming it applies to everyone.

**Example 1:**
> I lived off of potatoes almost exclusively for about 6 months. They were a dollar for 10 pounds, it was all I could afford. This was in Oregon about a decade ago.

**Example 2:**
> The only negative effect I've ever experienced is excessive urination, especially waking up at night to pee.
>
> This happens for me when I exceed about 1.5xbw in protein.

### The hardest boundary

**Which two labels:** `analysis` and `hot_take`. (A second, smaller boundary is `experience` vs `analysis` when a comment includes a number.)

**The decision rule I used every time:**

A comment is `analysis` only if it points to evidence someone else could look up: a link, or a named study, paper, book or statistic. Everything below that line is `hot_take`, even when it sounds expert or contains numbers:

- A number or percentage with no source named is `hot_take`.
- "Studies show…" or "they've found…" with no study named is `hot_take`.
- A credential or relationship ("I'm a dietitian", "my cousin is an RD", "source: my wife is an RD") is not evidence, so it is `hot_take`.

If a number or measurement describes the writer's own body or results (their bloodwork, weight, symptoms), it is `experience`, not `analysis`.

I wrote this rule in two steps. My first version said a comment needed "a checkable fact". After labelling my first 20 comments cold, I decided that a named source was the only thing I could apply the same way every time, and I tightened the rule. That changed two of my cold labels (see the hard cases below).

---

## The Dataset

**Where the posts came from:** Public Reddit comment threads about nutrition and healthy eating, 12 threads in all: several in r/nutrition (including threads on nutrition myths, "what do you wish everyone knew", "hottest nutrition takes", "underrated tips", and the Harvard sugar-sweetened-drinks study and protein side-effects threads), plus r/ScientificNutrition, r/science (an ultra-processed-food study), r/AskReddit (two threads), r/AskWomenOver40 (dietitians) and r/todayilearned (a potato-and-buttermilk diet). Most of the original posts were questions, so I collected the replies instead. I copied whole comments by hand from the thread pages and removed usernames and vote counts. I skipped jokes, plain questions, one-word replies, deleted comments, ads and off-topic tangents, because none of my three labels covers them. Everything is public, with no login needed.

**How I labelled them:** I labelled the first 20 comments cold, with no tools or help, and marked them `cold` in the note column. My original cold labels are saved in `cold_batch.csv`. The other 169 comments were pre-labelled by an AI tool (Claude Code) using my definitions and my rule, and marked `pre-labelled` in the note column. I then read every pre-labelled comment and marked it `pre-labelled, checked`, or `pre-labelled, corrected` if I changed the label. See "How I Used AI" for the disclosure.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| hot_take | 89 | 47.1% |
| experience | 63 | 33.3% |
| analysis | 37 | 19.6% |
| **Total** | 189 | 100% |

I stopped at 189 rows, which is under the 200 target and above the 150 floor, because I had run out of time to collect and review more. `analysis` is my thinnest label, and no label is above 60% of the file.

**Three hard cases**

**1.**
> *The post:* "This is called the “healthy user bias” and is relevant in almost all epidemiological studies. You have to see what they controlled for to deem it relevant, but with these studies you really can only show correlations."
>
> *Could have been:* `analysis` or `hot_take`.
>
> *I chose `hot_take`, because:* on my first, cold pass I labelled it `analysis`, since it names a real concept. Later I tightened my rule so that `analysis` needs a named, findable source, and this comment names a concept but no source. I changed it, and changed my other cold `analysis` comment that had no source (the "premature mortality increases BY 31%" comment) for the same reason.

**2.**
> *The post:* "Also, in regards to the context, they've found that people absorb more nutrients from their food if it's like...a comfort food? So if you were raised eating lasagne and you LOVE lasagne, you should try making that more healthy... (if I'm understanding it correctly)..."
>
> *Could have been:* `analysis` or `hot_take`.
>
> *I chose `hot_take`, because:* it claims "they've found" research but names no study. The same user later admitted she couldn't find the source. An unnamed study is not evidence under my rule.

**3.**
> *The post:* A comment that pastes an AI answer ("ChatGPT - Please comment on the supplement industry funding of research…") followed by a "Sources:" list of three papers with links.
>
> *Could have been:* `analysis` or `hot_take`.
>
> *I chose `analysis`, because:* it names three papers with links, which meets my rule on its face. I'm unsure about it, because AI-generated citations can be wrong and I did not check them, so "named" and "real" are not the same thing. It is the case that makes me least sure my rule is complete.

---

## The Training Run

**Base model:** `distilbert-base-uncased` (the notebook's default).

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42. Trained on CPU (torch 2.14.1, Python 3.12.3). The metrics are saved in `results.json`, and the held-out test posts in `test_split.csv`.

**Anything I changed from the defaults, and why:** Nothing. I used the notebook defaults for every setting.

**Split sizes:** 131 train, 29 validation, 29 test. Test split by label: `analysis` 6, `hot_take` 13, `experience` 10. `analysis` has fewer than 8 test examples, so its score will move a lot between seeds.

---

## How I Used AI

I used Claude Code (an AI coding assistant) throughout this project.

**Moment 1**

- *What I asked for:* After reading threads, I had three or four distinctions I could point at in the text (personal experience posts, replies sharing results, and replies that cited numbers or studies). I asked the AI to help me turn them into labels.
- *What came back:* Draft definitions for three labels, `analysis`, `hot_take` and `experience`, plus a suggested rule for the hard boundary (a number about the writer's own body is `experience`).
- *What I changed:* I chose the three labels and kept the "own body" rule, but I made the main rule stricter myself. I decided that `analysis` needs a named source, and that numbers, unnamed studies and credentials are all `hot_take`.

**Moment 2**

- *What I asked for:* I asked the AI to pre-label batches of comments using my definitions.
- *What came back:* It labelled comments such as "an egg has 1.5g of saturated fat, limit 10g/day" and "66–80% of people…" as `analysis` because they contain numbers, and flagged them as uncertain.
- *What I changed:* I told it that without evidence, a comment with numbers is still a `hot_take`, and that an unnamed study is a `hot_take`. It changed those rows and found others that broke the same rule, including two of my own cold labels, and I confirmed each change.

**Pre-labelling disclosure:**

I labelled 20 comments cold with no help. The other 169 comments were pre-labelled by Claude Code and marked `pre-labelled` in the note column. I read every one of those 169 and marked each `pre-labelled, checked` or `pre-labelled, corrected`. I changed 11 of the 169 pre-labels, all from `analysis` to `hot_take`, and all because of my named-source rule. I also used the AI to clean pasted thread text (removing usernames, vote counts and deleted comments), to set up the environment and notebook, and to suggest the target for criterion 5 in `criteria.md` (accuracy of at least 0.75 on the model's most-confident third). The other four targets and all the reasons in `criteria.md` are mine. This README was drafted with the AI from the facts of the project, and I edited it.

---

## Stretch Features

I declared these two stretch features before building them. I am not doing the fourth-label stretch.

1. **A second training run with one hyperparameter changed.** I will rerun the same 189 comments with the same seed (42) and everything else unchanged, but with `EPOCHS = 5` instead of 3. The metrics will be saved in `results_run2_epochs5.json`, and I will write down which numbers moved against the first run, and by how much.
2. **50 extra examples for my thinnest label, `analysis`.** `analysis` has 37 rows and only 6 test posts, and it scored F1 0.000 in my first run. I will collect at least 50 more comments that name a source, from new public Reddit threads, label and review them the same way as the rest, and retrain with the default settings. I will write down where they came from and what happened to `analysis` F1.

*(Both will be filled in below once they are built.)*

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
