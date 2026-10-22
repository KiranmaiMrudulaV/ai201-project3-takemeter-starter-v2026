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
