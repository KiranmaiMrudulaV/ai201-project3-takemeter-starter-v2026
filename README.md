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

TakeMeter sorts public Reddit comments about healthy eating, nutrition, protein and sugar into three kinds, based on how the writer supports what they say: `analysis` (a claim with a named, findable source), `hot_take` (a confident claim with nothing checkable behind it) and `experience` (what happened to the writer personally). I fine-tuned a small model (`distilbert-base-uncased`) on 189 comments I collected and labelled (248 after my stretch feature), so it can tell a sourced argument from an unsupported opinion from a personal story. The community is "people who talk about healthy eating on Reddit", and the labels came from what I saw in the comments I read, not from a textbook.

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

A link or named source counts as `analysis` unless the thread itself showed that source does not support the claim. A claim that only pairs with that disputed paper, with nothing said about what the paper found, stays `hot_take`. For example, "Keto literally causes diabetes" with a link to a Nature paper is `hot_take`, because another commenter pointed out the paper doesn't say that, while a comment that quotes what that paper found is `analysis`.

I wrote this rule in two steps. My first version said a comment needed "a checkable fact". After labelling my first 20 comments cold, I decided that a named source was the only thing I could apply the same way every time, and I tightened the rule. That changed two of my cold labels (see the hard cases below).

---

## The Dataset

**Where the posts came from:** Public Reddit comment threads about nutrition and healthy eating. My first 189 comments came from 12 threads: several in r/nutrition (including threads on nutrition myths, "what do you wish everyone knew", "hottest nutrition takes", "underrated tips", and the Harvard sugar-sweetened-drinks study and protein side-effects threads), plus r/ScientificNutrition, r/science (an ultra-processed-food study), r/AskReddit (two threads), r/AskWomenOver40 (dietitians) and r/todayilearned (a potato-and-buttermilk diet). Most of the original posts were questions, so I collected the replies instead. I copied whole comments by hand from the thread pages and removed usernames and vote counts. I skipped jokes, plain questions, one-word replies, deleted comments, ads and off-topic tangents, because none of my three labels covers them. Everything is public, with no login needed. The extra 59 comments for my stretch feature came from 10 more r/nutrition threads, mostly ones where people post and argue about studies (pineapple enzymes, Layne Norton on carbs and fat, the SWAP-MEAT trial, skin and wrinkles, vegetarian mortality, the LDL consensus statement, calcium, artificial sweeteners, walnuts, and whole grains).

**How I labelled them:** I labelled the first 20 comments cold, with no tools or help, and marked them `cold` in the note column. My original cold labels are saved in `cold_batch.csv`. The other comments were pre-labelled by an AI tool (Claude Code) using my definitions and my rule, and marked `pre-labelled` in the note column. I then read every pre-labelled comment and marked it `pre-labelled, checked`, or `pre-labelled, corrected` if I changed the label. The 59 stretch comments have `(stretch)` in the note column. See "How I Used AI" for the disclosure.

**Counts per label:**

First run (`results.json`, 189 comments):

| Label | Count | Share |
|---|---|---|
| hot_take | 89 | 47.1% |
| experience | 63 | 33.3% |
| analysis | 37 | 19.6% |
| **Total** | 189 | 100% |

Final `labels.csv` (248 comments, after the stretch feature):

| Label | Count | Share |
|---|---|---|
| hot_take | 96 | 38.7% |
| analysis | 89 | 35.9% |
| experience | 63 | 25.4% |
| **Total** | 248 | 100% |

I collected 189 comments for the first run, which is under the 200 target and above the 150 floor. I then collected 59 more to give `analysis`, my thinnest label, more examples. While reviewing them I moved 7 of the 59 to `hot_take` under my rule, so 52 of them are extra `analysis` rows. No label is above 60% of the file.

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

`results.json` and `test_split.csv` are this first run, on the first 189 comments. Two later runs (my stretch features) are saved separately: `results_run2_epochs5.json` (5 epochs, same 189 comments) and `results_run3_extra_analysis.json` with `test_split_run3.csv` (default settings, all 248 comments, 38 test posts).

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

I labelled 20 comments cold with no help. The other 169 comments were pre-labelled by Claude Code and marked `pre-labelled` in the note column. I read every one of those 169 and marked each `pre-labelled, checked` or `pre-labelled, corrected`. I changed 11 of the 169 pre-labels, all from `analysis` to `hot_take`, and all because of my named-source rule. The 59 stretch comments were also pre-labelled by Claude Code (all as `analysis`), and I reviewed every one, changing 7 of them to `hot_take`. In total I changed 18 of the 228 pre-labelled comments, all from `analysis` to `hot_take`. I also used the AI to clean pasted thread text (removing usernames, vote counts and deleted comments), to set up the environment and notebook, and to suggest the target for criterion 5 in `criteria.md` (accuracy of at least 0.75 on the model's most-confident third). The other four targets and all the reasons in `criteria.md` are mine. This README was drafted with the AI from the facts of the project, and I edited it.

**Unit 6 moments**

**Moment 3**

- *What I asked for:* After the before run missed two of my criteria, I asked the AI how to think about why.
- *What came back:* A list of possible causes (too few `experience` examples, a boundary I applied inconsistently, a hard label pair, a shortcut the model learned, truncation), with statistics it computed from `labels.csv` (row counts, how often each label has a link, how often each label uses first-person words, and the 5-epoch result).
- *What I changed:* I picked two causes myself: too few `experience` examples, and examples that are narrow and incomplete. The AI pointed out that the link shortcut explains `analysis` and not the `experience` miss, so I narrowed that claim, and it is written that way above. I also kept the line about what I could not rule out, because my agreement check cannot test the `experience` boundary.

**Moment 4**

- *What I asked for:* More varied `experience` examples for the improvement.
- *What came back:* The AI could not open Reddit, so I pasted five threads. It picked 27 whole comments from them and showed them to me one at a time, with a one-line description of each and a lean on the first one.
- *What I changed:* I labelled each of the 27 myself: 26 `experience` and 1 `hot_take`. I did not use any AI label. The AI chose the candidates, so my labels only cover comments it picked, and I flag that as a selection bias above.

**Unit 6 disclosure:** The AI ran the notebook for the three-seed runs and the baseline on copies, restored the files the notebook overwrites, filled in the result tables from the JSON files, wrote the one notebook change (`EXTRA_CSV`), and drafted the README text for Unit 6, which I approved and edited. My own work is the 30 staff labels (saved and committed before I saw the answer key), the adjudication of the one disagreement, the labels on the 27 new comments, my predictions, and the choice of diagnosis.

---

## Stretch Features

I declared these two stretch features before building them. I am not doing the fourth-label stretch.

1. **A second training run with one hyperparameter changed.** I will rerun the same 189 comments with the same seed (42) and everything else unchanged, but with `EPOCHS = 5` instead of 3. The metrics will be saved in `results_run2_epochs5.json`, and I will write down which numbers moved against the first run, and by how much.
2. **50 extra examples for my thinnest label, `analysis`.** `analysis` has 37 rows and only 6 test posts, and it scored F1 0.000 in my first run. I will collect at least 50 more comments that name a source, from new public Reddit threads, label and review them the same way as the rest, and retrain with the default settings. I will write down where they came from and what happened to `analysis` F1.

### Stretch 1 result: second run, `EPOCHS = 5`

The only change was `EPOCHS = 3` → `EPOCHS = 5`. Same 189 comments, same seed (42), same split (the test posts are identical), same CPU. First run: `results.json`. Second run: `results_run2_epochs5.json`.

| Measure | Run 1 (3 epochs) | Run 2 (5 epochs) | Change |
|---|---|---|---|
| Accuracy | 0.621 (18 of 29) | 0.724 (21 of 29) | +0.103 |
| Macro F1 | 0.468 | 0.675 | +0.207 |
| F1, `analysis` (6 test posts) | 0.000 | 0.500 | +0.500 |
| F1, `hot_take` (13 test posts) | 0.621 | 0.692 | +0.071 |
| F1, `experience` (10 test posts) | 0.783 | 0.833 | +0.050 |
| Accuracy, most-confident third | 0.889 | 1.000 | +0.111 |
| Accuracy, least-confident third | 0.222 | 0.556 | +0.333 |

What moved: with 3 epochs the model never predicted `analysis` (all 6 test `analysis` posts were called `hot_take`). With 5 epochs it got 2 of the 6 right, which is where almost all of the gain came from. `experience` went from 9 to 10 of 10 correct, and `hot_take` stayed at 9 of 13, with 4 still called `experience`. The model is now more confident when it is right.

How much I trust this: it is one seed and 29 test posts, so each post is worth about 3.4 points of accuracy. Three more posts correct is the whole difference in accuracy, and two of them are `analysis` posts. I can't tell yet from one run whether 5 epochs is better or whether this seed got lucky. Next unit's three seeds are what will show that.

### Stretch 2 result: extra `analysis` examples

I added 59 comments aimed at `analysis`, all of them naming or linking a source, from 10 new r/nutrition threads. After my review, 52 are `analysis` (above the 50 I declared) and 7 are `hot_take`. That took `analysis` from 37 to 89 comments. I retrained with the default settings (3 epochs, seed 42) on all 248 comments. Third run: `results_run3_extra_analysis.json`.

| Measure | Run 1 (189 comments, 29 test posts) | Run 3 (248 comments, 38 test posts) | Change |
|---|---|---|---|
| Accuracy | 0.621 (18 of 29) | 0.789 (30 of 38) | +0.168 |
| Macro F1 | 0.468 | 0.768 | +0.300 |
| F1, `analysis` | 0.000 (6 test posts) | 0.880 (13 test posts) | +0.880 |
| F1, `hot_take` | 0.621 (13 test posts) | 0.800 (15 test posts) | +0.179 |
| F1, `experience` | 0.783 (10 test posts) | 0.625 (10 test posts) | −0.158 |
| Accuracy, most-confident third | 0.889 | 1.000 | +0.111 |
| Accuracy, least-confident third | 0.222 | 0.667 | +0.445 |

What moved: `analysis` went from never being predicted correctly (0 of 6) to 11 of 13 correct, which is almost all of the improvement. `experience` got worse: 5 of its 10 test posts were called `hot_take`.

How much I trust this, and why it is not a clean comparison:
- The test set changed. Run 3 has 38 test posts, not 29, and 13 of them are `analysis`, so the two runs are not scored on the same posts.
- The new `analysis` comments are mostly long, link-heavy comments from science threads, which may be easier for the model to recognise than my earlier, shorter `analysis` comments. Part of the gain could be that the data became more uniform for this label, not only that there was more of it.
- `experience` fell from 33% to 25% of the data, which may be why it got worse. That is a guess, and I have not tested it.
- It is still a single seed.

Next unit's three seeds will show how much of this holds up.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both models were scored on the same 29 posts in `test_split.csv`, the held-out split from my first training run (`results.json`: seed 42, trained on my first 189 comments). The baseline is `facebook/bart-large-mnli` (zero-shot, no training) using my three label definitions from `label_definitions.txt`. It ran on CPU. Its predictions are in `baseline_results.json`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.241 | 0.621 | +0.379 |
| Macro F1 | 0.178 | 0.468 | +0.290 |
| F1 — `analysis` | 0.353 | 0.000 | -0.353 |
| F1 — `experience` | 0.182 | 0.783 | +0.601 |
| F1 — `hot_take` | 0.000 | 0.621 | +0.621 |

The baseline's confusion matrix on those 29 posts (rows are true labels, columns are its guesses):

| true \ predicted | analysis | hot_take | experience |
|---|---|---|---|
| **analysis** | 6 | 0 | 0 |
| **hot_take** | 13 | 0 | 0 |
| **experience** | 9 | 0 | 1 |

It called 28 of the 29 posts `analysis` and never predicted `hot_take`. For reference, always guessing `hot_take` would have scored 13 of 29 = 0.448, so the baseline did worse than ignoring the post.

**What I predicted before I looked:**

I predicted, before running the baseline, that the zero-shot model would fumble on `analysis` and `hot_take`, and that it would mix those two up with each other (calling `analysis` posts `hot_take` and `hot_take` posts `analysis`).

**Baseline with bare label names (no definitions):** I also ran the baseline once with no `label_definitions.txt`, so the model only sees the words `analysis`, `hot_take` and `experience`. Same 29 posts, same model, on CPU. Results are in `baseline_results_bare_names.json`.

| Measure | Baseline (my definitions) | Baseline (bare names) | Trained (run 1) |
|---|---|---|---|
| Overall accuracy | 0.241 | 0.621 | 0.621 |
| Macro F1 | 0.178 | 0.462 | 0.468 |
| F1 — `analysis` | 0.353 | 0.000 | 0.000 |
| F1 — `hot_take` | 0.000 | 0.688 | 0.621 |
| F1 — `experience` | 0.182 | 0.700 | 0.783 |

The bare-names baseline's confusion matrix (rows are true labels, columns are its guesses):

| true \ predicted | analysis | hot_take | experience |
|---|---|---|---|
| **analysis** | 0 | 5 | 1 |
| **hot_take** | 0 | 11 | 2 |
| **experience** | 0 | 3 | 7 |

It never predicted `analysis`. The baseline in the first table is the one the brief describes (my definitions as the prompt). The bare-names run turned out to be the stronger opponent, so I report both.

**What the gap actually means:**

The result depends on which baseline I use. Against the baseline with my definitions, my trained model wins by a lot (accuracy 0.621 vs 0.241), but that baseline called 28 of 29 posts `analysis` and scored below always guessing `hot_take` (0.448), so beating it is not much of a test. Without my definitions, the zero-shot model got 0.621 accuracy, exactly what my trained model got, and macro F1 of 0.462 vs 0.468. Against that stronger baseline my first fine-tuning run added essentially nothing on accuracy. The trained model was better on `experience` (0.783 vs 0.700) and slightly worse on `hot_take` (0.621 vs 0.688), and both scored 0.000 on `analysis`.

Three things I learned. First, my definitions made the zero-shot model much worse (0.241 with them, 0.621 without), so what they were worth for this model was negative. My guess, which I have not tested, is that this kind of model reads each long definition as a sentence to match, and the `analysis` definition, which talks about links and studies, matches almost any informational post. Second, this comparison is on only 29 posts, so each post is worth about 3.4 points of accuracy and a tie is within noise. Third, my trained model improved after this first run: with 5 epochs on the same 29 posts it scored 0.724 (accuracy) and 0.500 on `analysis` (`results_run2_epochs5.json`), which would beat the bare-names baseline by 0.103. That run changed a hyperparameter, so I am not using it as the main comparison.

My prediction (the baseline would fumble on `analysis` and `hot_take` and mix them up) was right for the baseline with my definitions on `hot_take` (F1 0.000, all 13 called `analysis`). For bare names it was wrong on `hot_take` (F1 0.688) and right only on `analysis`.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Trained with the default settings (3 epochs, learning rate 2e-5, batch size 16, 128 tokens) on all 248 comments in `labels.csv`, three random 70/15/15 splits (seeds 42, 7, 2024), each tested on 38 posts. Device: CPU (torch 2.14.1+cpu, Python 3.12.3). Results: `results_three_seeds_before.json`; the exact configuration is in `results_three_seeds_before_config.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy on the held-out test split | ≥ 0.60 on each seed | 0.789 | 0.579 | 0.763 | MISSED |
| 2. Every label's F1 (lowest label shown) | ≥ 0.50 on each seed | 0.625 (`experience`) | 0.000 (`experience`) | 0.625 (`experience`) | MISSED |
| 3. Largest label's share of `labels.csv` | ≤ 60% | 38.7% (`hot_take`) | 38.7% | 38.7% | MET |
| 4. Agreement with the staff labels | ≥ 70% | 97% (29 of 30) | same check | same check | MET |
| 5. Accuracy on the most-confident third of predictions | ≥ 0.75 on each seed | 1.000 | 0.833 | 1.000 | MET |

Criteria 3 and 4 do not change with the seed: criterion 3 is a property of the file, and criterion 4 is one comparison against the staff set (see the Agreement Report). The full per-label F1 by seed:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.789 | 0.579 | 0.763 | 0.211 |
| Macro F1 | 0.768 | 0.456 | 0.750 | 0.312 |
| F1 — `analysis` | 0.880 | 0.769 | 0.870 | 0.111 |
| F1 — `hot_take` | 0.800 | 0.600 | 0.757 | 0.200 |
| F1 — `experience` | 0.625 | 0.000 | 0.625 | 0.625 |

Accuracy moved 0.211 across the three seeds, more than ten points, so my dataset is too small or too lopsided for a stable measure. Test posts per label are `analysis` 13, `hot_take` 15, `experience` 10.

### Confusion matrix

Seed 42 (the matrix the notebook prints). Rows are true labels, columns are the model's guesses.

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | experience |
|---|---|---|---|
| **analysis** | 11 | 1 | 1 |
| **hot_take** | 1 | 14 | 0 |
| **experience** | 0 | 5 | 5 |

**My biggest off-diagonal number, and what it means:**

5 real `experience` posts were called `hot_take`, and 0 `hot_take` posts were called `experience`. So the model has not learned the boundary between a personal account and an unsupported general claim, and it errs toward `hot_take`. On seed 7 this was total: all 10 `experience` test posts were called `hot_take`, and the model never predicted `experience` once.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy on the held-out test split | ≥ 0.60 on all three seeds | **MISSED** | Accuracy was 0.789 / 0.579 / 0.763. Seed 7 is below 0.60, and the target has to hold on every seed. |
| 2 | Every label's F1 | ≥ 0.50 on all three seeds | **MISSED** | The lowest label was `experience` on every seed: 0.625 / 0.000 / 0.625. On seed 7 it is below 0.50 (the model never predicted it). |
| 3 | Largest label's share of `labels.csv` | ≤ 60% | **MET** | The largest is `hot_take`, 96 of 248 rows = 38.7%. |
| 4 | Agreement with the staff labels | ≥ 70% | **MET** | 29 of 30 posts = 97% (see the Agreement Report). |
| 5 | Accuracy on the most-confident third of predictions | ≥ 0.75 on all three seeds | **MET** | 1.000 / 0.833 / 1.000, all at or above 0.75 (12 predictions per third). |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criteria 1 and 2 (accuracy and every-label F1): one cause, `experience`.**

Both misses come down to the same label. In the before run `experience` had the lowest F1 on all three seeds (0.625 / 0.000 / 0.625). On seeds 42 and 2024, 5 of the 10 `experience` test posts were called `hot_take` and none went the other way. On seed 7 the model never predicted `experience` once, so all 10 were called `hot_take`, which is what pulled accuracy down to 0.579. My diagnosis is **too few examples of `experience`**, together with examples that are narrow, not varied enough, and not complete enough to show the boundary.

How I know:

- **Too few examples.** `experience` is my smallest label: 63 of 248 rows (25%), so about 44 in training and 10 in each test split. A label with 10 test posts swings a lot (each post is 10 points of its recall), which is how one seed could go to 0.000 while the other two stayed at 0.625.
- **The examples are narrow.** Only 4 of the 63 are my own cold labels. The other 59 were pre-labelled by the AI and checked by me, so they are in a narrower range of voices than real comments. By a simple word search, 16 of the 63 mix a personal story with a general-claim word (*everyone, always, never, you should*). Vegan or vegetarian stories (2 rows), keto or fat (3) and carbs (5) are thin. In a stretch run on the same data (`EPOCHS = 5`, first 189 comments), `experience` F1 went from 0.783 to 0.833, which fits a label that was under-trained and not hopeless.
- **It is not the staff-agreement kind of labelling problem.** My agreement with staff was 97%, and I agreed on 6 of the 7 borderline posts, so I apply a written rule consistently. I have to be honest about the limit of that: the staff taxonomy has `reaction`, not `experience`, so the agreement report cannot test my `experience` vs `hot_take` boundary. What I can say is that it weakens "my labelling is generally sloppy", and it leaves open that I applied my own `experience` boundary inconsistently. My improvement run (see below) is the evidence on that: when I labelled 27 new posts under the same rule and added them to training, `experience` F1 went up on every seed. If my boundary were inconsistent, more rows labelled under it would have added noise, not signal.
- **It is not a task the model can't reach.** The zero-shot baseline with bare label names got 7 of 10 `experience` posts right with no training at all, so the label can be found from this kind of text.
- **A shortcut explains `analysis`, not `experience`.** 72% of my `analysis` rows contain a link, against 3% of `hot_take` and 0% of `experience`, so the model has an easy cue for `analysis` (which scores 0.77–0.88). `experience` and `hot_take` have no such cue, since 37 of my 96 `hot_take` rows also use *I / my / me*. With few examples and no easy cue, the model fell back on the larger label, `hot_take`. I cannot claim the link shortcut caused the `experience` miss. It is my best explanation of why `analysis` looked fine while `experience` did not.

What I ruled out, and what I couldn't: I did not look at the individual misclassified `experience` posts (the notebook doesn't save per-post predictions), so I can't say whether the 5 errors were real boundary cases. That is a gap in my evidence.



---

## Agreement Report

I labelled the 30 staff posts under the staff taxonomy in `data/staff_taxonomy.md` (not my own), saved them as `my_staff_labels.csv`, and committed that file before I saw the staff answers. Then I compared them with `python agreement.py` (`agreement_results.json`).

**Agreement rate:** 29 / 30 = 97%

Staff flagged 7 of the 30 posts as borderline. I agreed with staff on 6 of those 7.

**Disagreements**

**1.**
> *The post:* "Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story and the sport has a problem."
>
> *Staff said / I said:* `hot_take` / `analysis`
>
> *My call, and why:* Staff is right. Rule 1 in the staff taxonomy says a post is `analysis` only if the checkable fact is load-bearing, and a number that is decoration on a conclusion the writer had already reached makes it a `hot_take` with a statistic in it. Here "so obviously the shoes are the whole story" is what does the work, and the 3% only decorates it. The post also gives nothing to check the number against (no year, no source), so nothing supports the jump from "times are down" to "the shoes are the whole story". Staff's own example, "Super shoes have ruined racing, times are down 3% since 2017", has the same shape. I read the 3% as a checkable stat and stopped there.
>
> *Which pile:* (a) the rule covered it and I applied it loosely.

**What the pattern in my disagreements tells me:**

There is only one disagreement, so I can't claim a pattern from this set alone. It is pile (a): the staff rule covered the case and I applied it loosely, by treating a bare number as support. That is the same kind of slip I made on my first, cold pass in Unit 5, when I labelled two comments `analysis` because they contained a statistic or named a concept, and only later tightened my rule to require a named source. I agreed with staff on 29 of 30 posts and on 6 of the 7 borderline ones, which suggests I apply a written rule fairly consistently. But this check cannot test the boundary my model struggles with most, `experience` vs `hot_take`, because the staff taxonomy has `reaction` and not `experience`. So it weakens "my labelling was generally inconsistent" as an explanation, but it does not rule out that I applied my own `experience` boundary inconsistently.

---

## The Improvement

**What I changed:**

I added 27 new hand-labelled comments to the **training data only**: 26 `experience` and 1 `hot_take`. They are in `labels_extra.csv`. Nothing else changed: same labels, same definitions, same `criteria.md`, same base model and training settings, same three seeds. I kept the new rows out of `labels.csv` on purpose. The notebook still draws each seed's split from `labels.csv` and then adds the extra rows to the training side only, so the validation and test posts are exactly the ones the before run used (38 test posts per seed, with an assertion in the notebook that no extra row appears in validation or test). That makes the before and after runs a controlled comparison.

Where the 27 came from: five public Reddit threads I pasted in (r/Protein, r/PetiteFitness, r/keto, r/Biohackers and r/AskFitnessIndia), whole comments only. The AI could not open Reddit itself, so it picked candidates from the threads I pasted. It chose comments it expected to be `experience`, which is a selection bias I should flag. I then labelled every one myself, one at a time: 26 `experience` and 1 `hot_take` (the short joking "protein farts" comment). They cover keto, plant-based and low-carb stories, symptoms, and routines. I did not manage to cover vegan or sugar stories, because none of the pasted threads had them. They were checked for duplicates against `labels.csv`.

**Which diagnosis pointed at it:**

The diagnosis for criteria 1 and 2: `experience` is the smallest label (about 44 training rows) and its examples are narrow. Adding more varied `experience` examples tests that directly. If the cause were my boundary, or the task being out of reach, extra rows should not have helped.

### Run Log — After

Same settings, same seeds, same test posts. Results: `results_three_seeds_after.json`; the configuration is in `results_three_seeds_after_config.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 on each seed | 0.842 | 0.763 | 0.763 | MET (was MISSED) |
| 2. Every label's F1 (lowest label shown) | ≥ 0.50 on each seed | 0.833 (`analysis`) | 0.690 (`hot_take`) | 0.690 (`hot_take`) | MET (was MISSED) |
| 3. Largest label's share of `labels.csv` | ≤ 60% | 38.7% | 38.7% | 38.7% | MET (`labels.csv` is unchanged; with the 27 extra rows, the largest share is `hot_take` at 97 of 275 = 35.3%) |
| 4. Agreement with the staff labels | ≥ 70% | 97% | same check | same check | MET |
| 5. Accuracy on the most-confident third | ≥ 0.75 on each seed | 0.917 | 1.000 | 1.000 | MET |

Before and after, side by side:

| Measure | Seed 42 | Seed 7 | Seed 2024 |
|---|---|---|---|
| Accuracy, before → after | 0.789 → 0.842 | 0.579 → 0.763 | 0.763 → 0.763 |
| Macro F1, before → after | 0.768 → 0.843 | 0.456 → 0.777 | 0.750 → 0.769 |
| F1 `experience`, before → after | 0.625 → 0.857 | 0.000 → 0.900 | 0.625 → 0.818 |
| Recall `experience`, before → after | 0.500 → 0.900 | 0.000 → 0.900 | 0.500 → 0.900 |
| F1 `analysis`, before → after | 0.880 → 0.833 | 0.769 → 0.741 | 0.870 → 0.800 |
| F1 `hot_take`, before → after | 0.800 → 0.839 | 0.600 → 0.690 | 0.757 → 0.690 |

Across the three seeds, mean accuracy went from 0.711 to 0.789, and the spread between seeds from 0.211 to 0.079.

Seed 42 confusion matrix after the improvement (rows are true labels, columns are guesses):

| true \ predicted | analysis | hot_take | experience |
|---|---|---|---|
| **analysis** | 10 | 2 | 1 |
| **hot_take** | 1 | 13 | 1 |
| **experience** | 0 | 1 | 9 |

**Did it help, and how do I know:**

Yes, for the label I was targeting. `experience` recall went from 5 of 10 (and 0 of 10 on seed 7) to 9 of 10 on all three seeds, and `experience` F1 rose on every seed by 0.19 to 0.90. Because the test posts are identical, that gain cannot come from an easier test set, and because it appears on all three seeds it is not a lucky draw. Criteria 1 and 2, which I missed before, are now met on all three seeds. I did not touch the targets in `criteria.md`.

What I would not claim:

- **It did not help everything.** `analysis` F1 fell on all three seeds (by 0.03 to 0.07), and `hot_take` F1 fell on seed 2024 (0.757 → 0.690). Real `hot_take` posts called `experience` went from 0 on every seed to 1, 1 and 3. Since 26 of my 27 new rows are `experience`, my best guess is that I nudged the model toward that label. That is a guess I have not tested.
- **The test sets are small.** Each test post is worth about 2.6 points of accuracy, and `experience` has only 10 test posts per seed. The pattern across seeds is more believable than any single number.
- **One change, one result.** It supports "too few `experience` examples" as the cause of the miss. It does not prove that the other causes I listed (narrow examples, no easy cue for `experience`) played no part, and the new rows came from five threads I picked, so they may be easier than a random sample of comments.

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

After the improvement, none of the five criteria is missed, but the model is not finished, and the criteria that are met have thin margins. The lowest accuracy is 0.763 (target 0.60), the lowest label F1 is 0.690 on `hot_take` (target 0.50), and `hot_take` is the label that is now weakest on two of the three seeds. What is still broken:

- **`hot_take` and `analysis` are blurry in both directions.** On seed 7, 4 real `hot_take` posts were called `analysis` and 3 real `analysis` posts were called `hot_take`. That is the pair I expected to be hard, and it still is. I did not try to fix it.
- **I may have pushed the model too far toward `experience`.** 3 real `hot_take` posts were called `experience` on seed 2024, up from 0. If I did another round I would add varied `hot_take` examples that read like personal stories (37 of my 96 `hot_take` rows use *I / my / me*), so the model learns the boundary and not just the label counts.
- **I could not check which test posts were wrong.** The notebook doesn't save per-post predictions, so I never read the individual `experience` posts the model got wrong. The next thing I would do is save them and read them.
- **Why I stopped:** the brief allows one improvement, I used it on the diagnosed cause, and with 38 test posts per seed another small change would be hard to tell from noise.



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

I meant `experience` to be anything where the writer reports what happened to them without claiming it applies to everyone, but the model read the label as "not an obvious source-backed post and not an obvious claim", and my first matrix shows it: 5 real `experience` posts were called `hot_take` and 0 went the other way (all 10 on seed 7). It only started to see my boundary after I added 26 more varied personal accounts, which tells me my rule was clear to me but my 63 examples did not show it fully.



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
