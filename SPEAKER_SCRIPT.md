# CRISPR-Net — speaker script

Companion to `crispr_net_talk.html`. The slides carry headlines and bullets only;
the explanation lives here.

**Budget: 9:40 across 20 slides.** Slide 21 is the thank-you slide, and 22–25 are backup, not in the clock, jump to
them only if asked. The deck has the same notes built in: press **N** for the presenter
panel, which also runs an elapsed clock (**R** resets it).

Audience: MLSN club members, mostly **not** from biology. Assume they know machine
learning and nothing about DNA.

**The one framing to protect all the way through:** this was a *reproduction*. We were not
trying to beat the paper, and we did not. Saying so plainly is the strongest thing we have.

---

## 1 — Title · 0:10

Hello, we're Group 2. Eight minutes: what CRISPR is, the dataset, the three tiers of
models we built, and what the numbers actually mean.

Say up front that this was a reproduction, so nobody spends the talk waiting for a win.

---

## 01 — CRISPR (≈2:50)

### 2 — DNA recap · 0:40

Start from zero; most of the room is not biology.

DNA is a text written in four letters, A, C, G, T. About three billion of them in every
one of your cells. It comes in two strands, and the second is completely determined by the
first: A always pairs with T, C always with G.

Point at the model on the left, that is the actual shape of the molecule, and the rungs
between the two strands are those pairs.

Finish on the pairing rule, because it sets up the next slide: a short piece of RNA can
find one exact address out of three billion, because it only sticks where the letters
match.

### 3 — How CRISPR cuts · 0:55

**Background first, two sentences, no more.** CRISPR is a bacterial immune system:
bacteria keep fragments of viruses they have survived and use them to recognise the virus
if it returns. In 2012 Jinek, Charpentier and Doudna showed the system could be
reprogrammed to cut *any* chosen DNA sequence, and that won the 2020 Nobel Prize in
Chemistry.

**Why the room should care:** it made targeted genome editing cheap and routine. An
approved sickle-cell therapy, engineered crops, diagnostics, and it is now standard
equipment in molecular biology labs.

**Then the mechanism, let the animation do the work.** The guide RNA scans along the DNA,
testing positions. When the letters match it locks on and the target lights up. Then Cas9
cuts, three letters in from the PAM.

The PAM is the constraint worth naming: a three-letter tag (NGG) just past the target. No
PAM, no cut, so you cannot target anywhere you like. It matters later because it is a
column in the data.

### 4 — The problem · 0:40

**This is the slide that turns biology into machine learning. Slow down here.**

Both of these are real rows from the dataset. Same length, same chemistry, both perfectly
valid guides. One cuts about five times more often than the other.

Nothing in the sequence makes that obvious to a human, and that is exactly the point.

And no simple rule predicts it, which is why this is a learning problem rather than a
lookup.

Then read the problem statement off the slide and move on: given only the 21-nucleotide
sequence, predict the cleavage efficiency, so candidates can be ranked computationally
instead of screened experimentally.

### 5 — Our project · 0:40

What we did this quarter: take a 2019 Nature Communications paper, rebuild its models from
the published description alone, and find out whether we land on the same answer.

**Emphasise: we worked from the paper, not from their code.** We never opened their
repository. Every model here is our own implementation.

Three things. *Reproduce*, rebuild the model ladder and compare. *Extend*, add models the
paper never tried. *Measure*, work out how much our own number wobbles before claiming
anything about theirs. The third one is the part we are most pleased with, and it comes
back at the end.

If there is time: it is one notebook start to finish, runs on Colab, resumes after a
disconnect, and 294 automated tests guard the split and the data.

---

## 02 — Data (≈1:05)

### 6 — The dataset · 0:25

This is what one row looks like. Read it left to right.

`gRNA_Seq` is the 20-letter guide. `PAM` is the three-letter tag we just saw. The `21mer`
is what we actually feed the model, the guide plus the first letter of the PAM.

The last three columns are the measured indel rates, how often the cut actually happened,
for three different versions of Cas9. Those three are our prediction targets. We used the
first one.

### 7 — The paper · 0:20

Wang et al., Nature Communications, September 2019. Open access, and the supplementary data
is downloadable, which is how we have their numbers.

The three nucleases are three versions of the same protein, engineered to be more precise.
We ran WT-SpCas9, the original and most widely used one.

Their seven models are the ladder we set out to reproduce.

### 8 — The split · 0:20

Thirty seconds on hygiene, because every number later depends on it.

Zero duplicate sequences, so a train/test leak is structurally impossible, not merely
unlikely.

The split is frozen into the cleaned data file with seed 42. Every model in every tier sees
the identical partition. That is what makes the tier comparison a measurement rather than a
coincidence.

---

## 03 — Models (≈1:45)

### 9 — Three tiers of models · 0:30

The spine of the project. Three tiers, each adding exactly one capability.

Tier 1: hand-engineered features, no deep learning. Tier 2: let the model read the raw
sequence itself. Tier 3: give it the sequence *and* the biophysics.

The daggered models are our additions beyond the paper's seven.

### 10 — Tier 1: gradient-boosted trees · 0:25

The cheap tier. No GPU, trains in minutes.

We turn each guide into 439 numbers: letter counts, position-by-position identity, GC
content, k-mer counts, melting temperature. Then hand them to tree models.

**The headline here:** our XGBoost scored 0.8454. The paper's XGBoost scored 0.8454,
matched to four decimal places, and on *less* training data than they used. That is the
moment we believed the pipeline was right.

The ensemble, a mean of the three boosters, was our best Tier-1 model at 0.8467.

### 11 — Tier 2: sequence-only deep models · 0:20

Tier 2 removes the hand-engineering entirely. The model gets the raw 21 letters and has to
work out for itself which positions and patterns matter.

The CNN slides filters along the sequence looking for local motifs. The recurrent models
read it in order, carrying context along.

Our CNN did well, 0.8548, better than every Tier-1 model. Our recurrent models did not.
Flag that and move on; slide 18 explains exactly why.

### 12 — Tier 3: sequence + biophysics · 0:30

Tier 3 is where the paper's best model lives, and ours too.

We add roughly 100 biophysical descriptors: RNA secondary-structure free energy from
ViennaRNA, how tightly the guide folds back on itself, plus melting temperatures for the
guide and its sub-regions, and GC content. Things the letters imply but do not state.

It costs 65 minutes of single-threaded CPU folding per nuclease before a single epoch of
training. The GPU does not help.

**The chart is the point of the tier.** The same features added +0.0120 to our CNN and
+0.0115 to the paper's RNN, the effect reproduces almost exactly, on a different
backbone. They did nothing for our BiLSTM (−0.0019), which is the tuning problem showing
up again.


---

## 04 — Results (≈3:00)

### 13 — The metric · 0:30

Before any result: what the score means. Mostly ML people here, so this is quick, but the
ordering point matters.

Spearman converts both lists to **ranks**, 1st, 2nd, 3rd, and correlates the ranks, not
the values. The slide shows the ranks explicitly under each row; point at them.

**Left:** the predicted values are all wrong, but they rank the three guides in the same
order as the experiment, so ρ = 1.

**Right:** the predicted values are almost exactly right, but two guides are ranked the
wrong way round, so ρ falls to 0.5.

Why it is the right metric here: a biologist orders the candidate guides for a gene and
tests the top few. Only the ordering determines what gets tested.

**Honest caveat:** it says nothing about whether the predicted numbers themselves are
trustworthy. Our R² is 0.71–0.80, quote that if someone asks.

### 14 — Our best model vs the paper's · 0:25

Say it plainly and do not oversell it.

Our best model scores 0.8668. Their best scores 0.8670, plus or minus 0.0024 across their
own ten runs.

We are two ten-thousandths below their mean, comfortably inside the range their own repeats
cover. That is what reproducing a result looks like.

**Say explicitly: we did not beat it.** Matching is not beating, and matching was the goal.

### 15 — Scoreboard · 0:25

All ten models, same held-out test set, coloured by tier.

Worth pointing at: Ridge at the bottom, a linear model on these features really is much
worse. The three boosters cluster tightly. CNN+bio on top.

And notice the two Tier-3 bars are far apart: CNN+bio is our best, BiLSTM+bio is near the
bottom. Same tier, same features. That is the tuning gap, not the tier.

### 16 — Tier by tier · 0:20

The ladder worked. Each tier beat the one below it.

Best Tier 1 to best Tier 2: +0.008. Tier 2 to Tier 3: +0.012.

Because the schedule was frozen, those gaps are attributable to the tier rather than to
tuning. So on this problem, biology was worth more than depth.

### 17 — Against the paper · 0:30

**This is the slide I would most like people to take away. Give it time.**

We pulled the paper's own per-fold supplementary data, ten runs per model, so these are
exact comparisons, not values read off a figure.

The grey band is the full range their ten runs covered. The vertical line is their
published mean. The coral dot is us.

- XGBoost, CNN+bio, Ridge: inside their range. Reproduced.
- CNN: above all ten of their runs. Interesting, but only 1.5σ once our own split noise is
  counted. Suggestive, not a result.
- BiLSTM: 4.7σ below their worst fold. That one is ours to own, and the next slide explains
  it.

### 18 — Why we differ · 0:30

Three reasons, and none of them is "we got it wrong".

1. They trained tree models on 85% with no validation set; we use 76.5% throughout. Our
   XGBoost saw *less* data and matched them anyway.
2. They selected features with Tree SHAP from a pool of 1,031; we feed all 439 with no
   selection step.
3. **Say this one out loud.** They tuned every model; we deliberately did not, so the tier
   comparison stays controlled. That is the entire BiLSTM gap, and we can prove it: a
   40-trial search on the same architecture reached 0.8637.

*(If someone asks about the error bars: theirs is ten different splits, ours was three
weight initialisations on one split. We measured the real split-to-split spread ourselves,
0.0053, and that is what the z-scores on slide 17 use. It is on backup slide 24.)*

---

## 05 / 06 — Close (≈1:35)

### 19 — Limitations and future work · 0:35

Keep this light; it is the last content slide before the summary.

**Limitations.** The honest framing is that we were a student team on free Colab GPUs for
one quarter. That is the reason behind all four, not a methodological failure. One split
instead of ten because ten costs ten times the compute. One nuclease because Tier 3 needs
an hour of CPU folding per nuclease before training even starts. And no MLP simply because
we ran out of time.

**Future work.** Build the models we did not reach, which covers both the paper's own RNN
and RNN + biofeatures and the Transformer, multi-task and stacked models already written in
our codebase. Then a full tenfold so our error bar measures the same thing theirs does, and
finally the other two nucleases.

### 20 — Summary · 0:30

The slide carries only what we established, so **say the honest half out loud**; it does
not need to be on screen:

> We did not beat the paper. Our CNN edge is not statistically real once our own split
> noise is counted, and our BiLSTM is genuinely below theirs. We know exactly where we
> fall short and why.

A clean slide plus a candid sentence lands better than a column of caveats nobody reads.

### 21 — Thank you · not in the clock

Leave this one up while taking questions. Everything after it is backup: 22 the frozen
training schedule, 23 the full scoreboard, 24 our split noise, 25 the tuning study.


---

## Backup slides (not in the clock)

**22, One schedule.** *(Archived from the main flow, no longer in the eight minutes, but worth having if anyone asks why the BiLSTM underperforms.)*

This is the methods choice behind the BiLSTM shortfall. It is also the answer to "why
didn't you just tune it?"

All five deep models train on the identical schedule. Nothing is special-cased.

So LSTM → BiLSTM is bidirectionality and nothing else. BiLSTM → BiLSTM+bio is the biology
features and nothing else.

The paper instead tuned every model separately with Bayesian optimisation. Both are valid;
they answer different questions. Ours buys clean tier comparisons, and it costs us the
BiLSTM.


**23, Full scoreboard.** The complete table with the paper's value and the delta where one
exists. LightGBM, CatBoost and Ensemble have no paper row because the paper never tried
them.

**24, Split noise.** We retrained XGBoost seven times, changing only which 15% is held out.
Nothing else moved. Spread 0.0175, nearly twice the margin our CNN appeared to beat the
paper by. That is why we will not claim the CNN result.

**25, Tuning study.** 40 Optuna trials on the BiLSTM+bio architecture. Learning rate and
weight decay together account for 91% of the variance; hidden size, layers and merge mode
account for almost nothing. On this problem, how you train matters far more than what you
train, which is why the BiLSTM gap is a tuning gap, not an architecture gap.

---

## Likely questions

**"Did you beat the paper?"**
No, and we are not claiming to. We matched their headline result. Our CNN is above all ten
of their CNN runs, but once we account for our own split-to-split noise that is only 1.5σ,
not enough to call a result.

**"Why is your BiLSTM so much worse?"**
Because we deliberately did not tune it. Every deep model runs on one frozen schedule so
the tier-to-tier comparisons stay controlled. A 40-trial search on the same architecture
reached 0.8637, so it is a tuning gap, not an architecture gap. (Backup slide 25.)

**"How do you know your number isn't just luck?"**
We measured it. Seven different splits of the same data move XGBoost's score by 0.0175.
That measured spread is what we use when judging any difference. (Backup slide 24.)

**"Why only Spearman?"**
It is the paper's headline metric, and it matches the use case, you pick the top few
guides, so only the ordering matters. Our R² is 0.71–0.80 if you want the calibration
number, which is the less flattering one.

**"Why only one nuclease?"**
GPU time. Tier 3 needs about 65 minutes of CPU RNA-folding per nuclease before training
starts, and that is paid whatever else you do. WT is the original and most widely used
Cas9, so it was the one to do first.

**"Did you use their code?"**
No. Everything is our own implementation from the published description. We used their
supplementary *data*, the dataset and their per-fold results, but never their source.
