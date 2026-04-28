# Poster Content: Comparative Error Analysis of POS Taggers

## Title

**Comparative Error Analysis of POS Taggers on UD English EWT**

Optional shorter version:

**Comparing Classical and Neural POS Taggers on UD English EWT**

## Authors / Affiliation

Replace with your final names and affiliation:

- `Author 1`
- `Author 2`
- `Innopolis University`

## One-Sentence Summary

We compare four part-of-speech tagging approaches on the same Universal Dependencies benchmark and show that the Compact BiLSTM achieves the best final quality, while CRF provides the strongest trade-off between accuracy, speed, and interpretability.

## Motivation

Part-of-speech tagging is a standard sequence labeling problem and a useful testbed for comparing classical and neural NLP models under the same evaluation protocol. The task is simple enough to analyze in detail, but rich enough to reveal meaningful differences in how models behave on ambiguous words, long sentences, and out-of-vocabulary items.

This case study is motivated by three goals:

- compare several model families on the same benchmark
- move beyond final accuracy and analyze *why* models fail
- provide a practical recommendation, not only a ranking

## Research Question

Which POS tagging model gives the best balance between:

- final quality
- robustness to difficult inputs
- training and inference cost
- interpretability

## Task Definition

Given a sentence tokenized into words, predict a POS tag for every token using the Universal POS (`UPOS`) label set.

This study uses **17 POS tags** from Universal Dependencies.

## Dataset

The experiments use **UD English EWT** with the original fixed `train/dev/test` split.

### Dataset Statistics

| Split | Sentences | Tokens | Avg sent. length | Median sent. length | Max sent. length | Vocabulary size | Token OOV rate |
|---|---:|---:|---:|---:|---:|---:|---:|
| Train | 12,544 | 204,577 | 16.309 | 14.0 | 159 | 19,674 | - |
| Dev | 2,001 | 25,147 | 12.567 | 10.0 | 75 | 5,494 | 0.08303 |
| Test | 2,077 | 25,094 | 12.082 | 9.0 | 81 | 5,629 | 0.09134 |

Average sentence-level OOV rates:

- Dev: `0.12843`
- Test: `0.13837`

## Methodology

All models are trained and evaluated under the same conditions:

- same train/dev/test split
- same tokenization
- same sentence boundaries
- same `UPOS` tag inventory
- same evaluation protocol

For every model we report:

- token accuracy
- macro-F1
- training time
- inference time
- tokens per second
- estimated model size
- per-tag precision, recall, and F1
- confusion matrix
- error rate by sentence length
- error rate by OOV bucket
- representative qualitative error examples

## Models

### 1. Averaged Perceptron

A feature-based linear baseline trained with averaged online updates.

Main features:

- current word and lowercase form
- prefixes and suffixes
- word shape
- capitalization and digit patterns
- neighboring words
- previous predicted tags

Why include it:

- strong classical baseline
- easy to implement
- easy to interpret

### 2. HMM + Viterbi

A probabilistic sequence model using:

- transition probabilities between tags
- emission probabilities from tags to words
- Viterbi decoding for the best global tag sequence

Why include it:

- highly interpretable
- smallest model
- near-instant training

### 3. CRF

A feature-based conditional random field that models the entire output tag sequence conditioned on the sentence.

Why include it:

- strong classical sequence model
- usually stronger than HMM on contextual decisions
- very interpretable relative to neural alternatives

### 4. Compact BiLSTM

A neural baseline with:

- word embeddings
- character-level encoding
- bidirectional LSTM sequence encoder
- linear token classifier

Why include it:

- neural comparison point
- contextual sequence modeling
- stronger generalization than local feature methods

## Experimental Pipeline

The experimental flow is:

1. Load the benchmark from raw `.conllu` files.
2. Build the training vocabulary and label set.
3. Train each model on the same training split.
4. Evaluate on dev and test.
5. Compare global metrics.
6. Analyze failures through confusion matrices, OOV buckets, sentence-length buckets, and qualitative error examples.

## Development Results

| Model | Dev accuracy | Dev macro-F1 | Train time (s) | Inference time (s) | Tokens/sec | Model size (MB) |
|---|---:|---:|---:|---:|---:|---:|
| Compact BiLSTM | 0.952042 | 0.898645 | 574.5724 | 1.2176 | 20,652.61 | 11.9991 |
| CRF | 0.945838 | 0.896580 | 22.9582 | 0.1419 | 177,238.47 | 4.6725 |
| Averaged Perceptron | 0.944407 | 0.879269 | 32.7098 | 0.6529 | 38,515.07 | 14.6633 |
| HMM + Viterbi | 0.905158 | 0.828556 | 0.1010 | 2.5156 | 9,996.28 | 0.6275 |

## Final Test Results

| Model | Test accuracy | Test macro-F1 | Train time (s) | Inference time (s) | Tokens/sec | Model size (MB) |
|---|---:|---:|---:|---:|---:|---:|
| Compact BiLSTM | **0.952419** | **0.904233** | 574.5724 | 1.2011 | 20,891.80 | 11.9991 |
| CRF | 0.946880 | 0.899161 | 22.9582 | **0.1398** | **179,476.44** | 4.6725 |
| Averaged Perceptron | 0.943652 | 0.894724 | 32.7098 | 0.6227 | 40,296.88 | 14.6633 |
| HMM + Viterbi | 0.911294 | 0.840027 | **0.1010** | 2.8858 | 8,695.58 | **0.6275** |

## Main Quantitative Findings

- **Compact BiLSTM** is the best model by final quality.
- **CRF** is the best classical model and the best overall trade-off.
- **Averaged Perceptron** remains surprisingly competitive.
- **HMM + Viterbi** is the weakest model by quality, but also the cheapest to train and smallest to store.

## Beyond Raw Scores: What The Differences Mean

The absolute metric gaps are not very large between the top three models, but their computational costs are very different.

### Compact BiLSTM vs CRF

- Accuracy gain: **+0.554 percentage points**
- Macro-F1 gain: **+0.507 percentage points**
- Training cost: **25.0x higher**
- Inference time: **8.6x slower**
- Model size: **2.57x larger**

Interpretation:

The neural model is clearly best by quality, but the gain over CRF is relatively modest compared with its much higher computational cost. This makes the result interesting rather than trivial: the best model is not automatically the best recommendation.

### CRF vs Averaged Perceptron

- Accuracy gain: **+0.323 percentage points**
- Macro-F1 gain: **+0.444 percentage points**
- Training time: **lower than the perceptron run in this setup**
- Inference speed: **about 4.45x higher throughput**
- Model size: **about 3.1x smaller**

Interpretation:

CRF improves both accuracy and efficiency relative to the perceptron in this implementation. This is why it becomes the strongest classical option, not just the most accurate one.

### Averaged Perceptron vs HMM

- Accuracy gain: **+3.236 percentage points**
- Macro-F1 gain: **+5.470 percentage points**

Interpretation:

This gap is large enough to show that explicit feature engineering is much more useful here than relying only on simple transition/emission statistics.

### A non-obvious finding: size alone does not explain quality

The results do not follow a simple “bigger model is always better” pattern:

- HMM is the smallest model and clearly the weakest
- Perceptron is larger than CRF in serialized size, but weaker
- CRF is much smaller than BiLSTM, yet very close in quality

This means the most important factor is not raw model size, but how well the model represents contextual ambiguity.

## Error Analysis

### Most Common Confusions

The main confusion across all models is:

- `PROPN -> NOUN`

The reverse direction is also consistently common:

- `NOUN -> PROPN`

This means that distinguishing proper nouns from common nouns remains the hardest recurring boundary in the benchmark.

### Top Confusions on the Test Split

| Model | Most common confusions |
|---|---|
| Averaged Perceptron | `PROPN -> NOUN` 224, `NOUN -> PROPN` 184, `VERB -> NOUN` 78 |
| HMM + Viterbi | `PROPN -> NOUN` 375, `NOUN -> PROPN` 161, `VERB -> AUX` 105 |
| CRF | `PROPN -> NOUN` 210, `NOUN -> PROPN` 174, `VERB -> NOUN` 70 |
| Compact BiLSTM | `PROPN -> NOUN` 218, `NOUN -> PROPN` 158, `VERB -> NOUN` 59 |

### Interpretation of Confusions

- Proper nouns depend strongly on context, casing, and lexical familiarity.
- HMM shows an especially strong `VERB -> AUX` error pattern, which matches its dependence on transition statistics.
- CRF and BiLSTM reduce many classical mistakes, but neither removes the `PROPN <-> NOUN` ambiguity.

## Deeper Error Interpretation

### 1. The hardest problem is not random noise but a stable linguistic boundary

The dominant `PROPN <-> NOUN` pattern appears in every model, including the strongest one. This means the main challenge is not just insufficient optimization, but a genuinely difficult linguistic distinction. Proper-name recognition often depends on discourse context, lexical familiarity, and subtle capitalization cues that are not always decisive.

### 2. The HMM errors reveal a modeling limitation, not just weaker training

HMM does not only score lower overall. Its error profile is qualitatively different:

- much larger `PROPN -> NOUN` confusion
- strong `VERB -> AUX` confusion
- weaker low-frequency tag handling

This suggests that the model fails in exactly the places where fixed emission/transition statistics are too rigid.

### 3. CRF improves sequence decisions without giving up interpretability

CRF beats the perceptron by a small but consistent margin while staying far cheaper than the neural model. The likely reason is not better local features, but better global sequence consistency. In other words, CRF benefits from structured prediction rather than from a completely different feature space.

### 4. BiLSTM wins mostly by consistency, not by changing the error landscape

The Compact BiLSTM becomes the best model overall, but its top confusions still look very similar to the classical models. This is an important result: the neural model improves accuracy mainly by reducing the frequency of familiar mistakes, not by solving a fundamentally different set of cases.

### 5. The benchmark seems to contain “hard pockets” rather than uniformly distributed noise

The same difficult sentences recur across notebooks, especially in:

- long examples
- OOV-heavy examples
- name-rich or web-style text

This suggests that model failures are concentrated in specific linguistic regimes rather than spread evenly across the corpus. In practice, this is useful: it means targeted improvements may have a clear payoff.

## Per-Tag Findings

The easiest tags across models are highly regular categories such as:

- `PUNCT`
- `CCONJ`
- `DET`
- `PRON`
- `AUX`

The hardest tags across models are:

- `X`
- `SCONJ`
- `PROPN`
- `SYM`
- `INTJ`

This suggests that model changes improve overall quality, but do not fundamentally change which linguistic categories are most difficult.

A useful poster-level interpretation is:

- easier tags are supported by strong local cues or punctuation regularity
- harder tags depend on broader context, rare lexical items, or annotation ambiguity
- model improvements mostly help frequent ambiguous categories, but they do not fully solve sparse or exceptional ones

### Closed-class vs open-class insight

Another useful interpretation is that the models almost solve many **closed-class categories** such as punctuation, determiners, pronouns, conjunctions, and auxiliaries. The real bottleneck is mostly in **open-class or heterogeneous categories**:

- nouns
- proper nouns
- adjectives
- adverbs
- miscellaneous tags such as `X` and `SYM`

This means that future improvements are unlikely to come from better handling of easy grammatical categories. They will come from stronger lexical generalization and better disambiguation of content words.

## Robustness Findings

The plots in the model notebooks show that:

- stronger models remain more stable on long sentences
- stronger models also remain more stable under higher OOV pressure
- HMM degrades most clearly on difficult inputs
- CRF and Compact BiLSTM are the most robust alternatives overall

Qualitative error examples confirm that many hard cases combine:

- long sentences
- many unseen words
- named entities or unusual lexical items

This matters because it shows that the benchmark is not only measuring memorization of common tokens. The hardest model failures come from exactly the kinds of inputs that stress generalization.

### Why OOV matters so much here

OOV pressure is especially informative in this study because POS tagging is often treated as a “solved” task on frequent tokens. Once unseen or rare lexical items appear, the models must fall back on:

- morphology
- local syntax
- contextual expectations

This is exactly where the gap between HMM and the stronger models becomes most meaningful.

## Cross-Model Insights

### Classical vs Neural

The study does not show a clean “neural always wins” story. Instead, it shows a layered result:

- BiLSTM is best by raw quality
- CRF is best by cost-quality balance
- perceptron remains a serious baseline
- HMM is mainly valuable for interpretability and pedagogy

### Why the ranking is meaningful

The ranking is meaningful because the models differ not only in score, but also in:

- training cost
- inference speed
- model size
- interpretability
- error profile

This makes the case study closer to a real model-selection problem and less like a leaderboard exercise.

### Why the classical models remain strong

A useful high-level conclusion is that POS tagging still rewards strong classical modeling because:

- many useful signals are local and structured
- orthographic and suffix cues are informative
- the label space is small and linguistically constrained

This is why CRF and even the perceptron remain highly competitive despite the availability of neural sequence models.

### Why the neural model still wins

At the same time, the BiLSTM gains enough quality to finish first because it can combine:

- bidirectional context
- character-sensitive lexical information
- distributed representations that generalize beyond exact memorization

So the final picture is not “classical is obsolete” or “neural always dominates”, but a more balanced result: classical methods remain very strong, while the neural model provides the best final accuracy if the added cost is acceptable.

## Limitations

The poster should also acknowledge the main limits of the study:

- only one benchmark is used
- the main reported runs use `UPOS`, not fine-grained `XPOS`
- no extensive hyperparameter search was performed
- the BiLSTM is still a compact neural model, not a large pretrained encoder
- the analysis is based on one implementation per model family rather than many ablations
- the poster conclusions are based on one benchmark domain, so domain transfer is not directly tested

These limitations are acceptable for a compact comparative study, but they explain why the conclusions should be read as practical rather than universal.

## Future Work

The strongest natural extensions are:

- evaluate on `XPOS` for finer-grained confusion analysis
- add character or subword ablations for the neural model
- compare CRF and BiLSTM on additional UD languages
- analyze whether pretrained embeddings further reduce `PROPN <-> NOUN` ambiguity
- test whether hybrid architectures such as BiLSTM-CRF change the error landscape

## Practical Recommendation

### Best model by raw quality

**Compact BiLSTM**

Why:

- highest test accuracy
- highest macro-F1
- strongest overall per-tag performance

### Best practical trade-off

**CRF**

Why:

- very close to the best model in quality
- much cheaper to train than BiLSTM
- much faster at inference
- still highly interpretable

Poster-friendly interpretation:

CRF is the model that most likely would be chosen in a real constrained setting where compute, speed, and explanation quality all matter.

### Best model for “almost-best quality with operational efficiency”

This is a separate and useful recommendation category:

**CRF is the dominant near-optimal model.**

Why this is important:

- it loses only a small amount of quality to BiLSTM
- it avoids the large computational penalty of the neural model
- it keeps the error profile understandable

### Best simple baseline

**Averaged Perceptron**

Why:

- strong results despite simple architecture
- easy to explain
- competitive with CRF in overall accuracy

### Best pedagogical baseline

**HMM + Viterbi**

Why:

- smallest model
- fastest training
- easiest probabilistic interpretation

## Final Conclusion

The case study shows that the best model depends on the constraint:

- if the goal is maximum final quality, choose **Compact BiLSTM**
- if the goal is the strongest balance between quality, speed, and interpretability, choose **CRF**
- if the goal is a simple and strong baseline, choose **Averaged Perceptron**
- if the goal is pedagogical clarity and minimal cost, choose **HMM + Viterbi**

The most important qualitative result is that all models struggle with the same core ambiguity: distinguishing **proper nouns** from **common nouns**, especially in longer or OOV-heavy sentences.

Another important conclusion is that most of the benchmark is *not* equally difficult. The strongest errors cluster in interpretable regions: long sentences, rare words, and content-word ambiguities. This makes the analysis more useful than a simple benchmark ranking because it identifies where future model improvements should focus.

## Strong Poster Takeaways

If you want stronger “message sentences” for the poster, these are the most useful:

1. **The best model by score is not the best model by practicality.**
2. **CRF nearly matches BiLSTM quality while being far cheaper and faster.**
3. **The main unresolved problem across all models is `PROPN <-> NOUN` ambiguity.**
4. **Hard errors concentrate in long, OOV-heavy, name-rich sentences.**
5. **The study shows a real trade-off between quality, efficiency, and interpretability.**
6. **Many easy grammatical categories are nearly solved; the real challenge is ambiguous content words.**
7. **Classical models remain strong because POS tagging still rewards structured local signals.**
8. **The neural model wins by reducing familiar mistakes, not by changing which mistakes exist.**

## Poster-Ready Text Blocks

### Very Short Motivation Block

We compare classical and neural POS taggers on the same benchmark to understand not only which model is best, but also where each model fails and which trade-offs matter in practice.

### Very Short Method Block

We train four POS tagging models on UD English EWT under the same train/dev/test split and compare them using token accuracy, macro-F1, runtime, confusion matrices, and robustness to long and OOV-heavy sentences.

### Very Short Results Block

Compact BiLSTM achieves the best final quality (`95.24%` accuracy, `0.904` macro-F1), while CRF is almost as accurate and much faster, making it the best practical trade-off.

### Very Short Conclusion Block

Neural sequence tagging gives the best raw score, but CRF remains the most balanced recommendation for a compact, explainable NLP project.

## Recommended Visuals For The Poster

If you later turn this into a visual poster, the most useful figures to include are:

1. A **dataset/setup box** with split sizes and OOV rates.
2. A **model overview box** with one short description per model.
3. A **main results table** with test accuracy, macro-F1, train time, and inference speed.
4. A **bar chart of error rate by sentence length**.
5. A **bar chart of error rate by OOV bucket**.
6. A **confusion matrix snapshot** for the best model or for CRF vs BiLSTM.
7. A **small box with top confusion pairs**.
8. A **final recommendation box** summarizing which model to choose under which constraint.
