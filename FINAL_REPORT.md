# Comparative Error Analysis of POS Taggers

Submission note:
The project is organized as a notebook-first repository. Data preparation and benchmark inspection are documented in `notebooks/01_data_and_benchmark.ipynb`. The four model experiments are split into separate notebooks: `02_averaged_perceptron.ipynb`, `03_hmm_viterbi.ipynb`, `04_crf.ipynb`, and `05_compact_bilstm.ipynb`. The main experiment artifacts are shown directly in notebook outputs rather than being stored as separate committed result files.

## 1. Project Goal

The goal of this case study is to compare four sequence labeling approaches for part-of-speech tagging on the same benchmark split:

- Averaged Perceptron
- HMM with Viterbi decoding
- CRF
- Compact BiLSTM

The project compares both final quality and model behavior under error analysis. The main questions are:

- which model achieves the best overall tagging quality
- which model offers the best practical trade-off between accuracy, speed, and interpretability
- which confusions and failure modes are most common across models

## 2. Experimental Setup

The experiments use the Universal Dependencies English EWT corpus with the original fixed `train/dev/test` split. All models are trained and evaluated on the same tokenization, the same sentence boundaries, and the same `UPOS` label inventory. This keeps the comparison fair.

### Dataset statistics

| Split | Sentences | Tokens | Avg sent. length | Median sent. length | Max sent. length | Vocabulary size | Token OOV rate |
|---|---:|---:|---:|---:|---:|---:|---:|
| Train | 12,544 | 204,577 | 16.309 | 14.0 | 159 | 19,674 | - |
| Dev | 2,001 | 25,147 | 12.567 | 10.0 | 75 | 5,494 | 0.08303 |
| Test | 2,077 | 25,094 | 12.082 | 9.0 | 81 | 5,629 | 0.09134 |

The benchmark notebook also shows average sentence-level OOV rates:

- Dev: `0.12843`
- Test: `0.13837`

### Evaluation protocol

Each model notebook reports the same output types:

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
- representative error examples

## 3. Models

### 3.1 Averaged Perceptron

This is a feature-based linear baseline trained with averaged online updates. It uses local lexical, orthographic, and contextual features.

Strengths:

- simple to implement
- strong baseline quality
- easy to explain

Weakness:

- depends on hand-crafted local features

### 3.2 HMM + Viterbi

This is a classical probabilistic sequence model based on transition and emission statistics with Viterbi decoding.

Strengths:

- highly interpretable
- tiny model size
- almost instant training

Weakness:

- more brittle on unseen words and ambiguous contexts

### 3.3 CRF

The CRF uses contextual feature engineering like the perceptron, but optimizes the conditional probability of the full tag sequence.

Strengths:

- strongest classical model in the project
- very fast at inference
- high interpretability

Weakness:

- still tied to manual feature design

### 3.4 Compact BiLSTM

This neural baseline uses word embeddings, character-level encoding, a bidirectional LSTM encoder, and a linear classifier. Although it is described as compact, in the current implementation it is still the strongest model by final quality.

Strengths:

- best accuracy and macro-F1 in the current runs
- strong contextual generalization
- strongest overall per-tag quality

Weakness:

- by far the slowest model to train
- less interpretable than the classical alternatives

## 4. Development Results

| Model | Dev accuracy | Dev macro-F1 | Train time (s) | Inference time (s) | Tokens/sec | Model size (MB) |
|---|---:|---:|---:|---:|---:|---:|
| Compact BiLSTM | 0.952042 | 0.898645 | 574.5724 | 1.2176 | 20,652.61 | 11.9991 |
| CRF | 0.945838 | 0.896580 | 22.9582 | 0.1419 | 177,238.47 | 4.6725 |
| Averaged Perceptron | 0.944407 | 0.879269 | 32.7098 | 0.6529 | 38,515.07 | 14.6633 |
| HMM + Viterbi | 0.905158 | 0.828556 | 0.1010 | 2.5156 | 9,996.28 | 0.6275 |

Interpretation:

- The Compact BiLSTM already leads on dev by token accuracy and macro-F1.
- CRF is very close in quality while being dramatically cheaper to train and much faster to run.
- Averaged Perceptron remains a strong baseline, especially considering its simplicity.
- HMM trains almost instantly, but the quality gap is already large on dev.

## 5. Final Test Results

| Model | Test accuracy | Test macro-F1 | Train time (s) | Inference time (s) | Tokens/sec | Model size (MB) |
|---|---:|---:|---:|---:|---:|---:|
| Compact BiLSTM | 0.952419 | 0.904233 | 574.5724 | 1.2011 | 20,891.80 | 11.9991 |
| CRF | 0.946880 | 0.899161 | 22.9582 | 0.1398 | 179,476.44 | 4.6725 |
| Averaged Perceptron | 0.943652 | 0.894724 | 32.7098 | 0.6227 | 40,296.88 | 14.6633 |
| HMM + Viterbi | 0.911294 | 0.840027 | 0.1010 | 2.8858 | 8,695.58 | 0.6275 |

Main conclusions from the final table:

- `Compact BiLSTM` is the best model by raw quality.
- `CRF` is the best non-neural model and the strongest practical trade-off.
- `Averaged Perceptron` is a credible and competitive simple baseline.
- `HMM + Viterbi` is useful as a teaching baseline, but not competitive by final quality.

## 6. Error Analysis

### 6.1 Dominant confusion patterns

The same broad confusion pattern appears across all four models:

- `PROPN -> NOUN`
- `NOUN -> PROPN`

This is the main recurring ambiguity in the benchmark. It appears in every model and is the single largest confusion count in all four notebooks.

Top confusion counts on the test split:

| Model | Most common confusions |
|---|---|
| Averaged Perceptron | `PROPN -> NOUN` 224, `NOUN -> PROPN` 184, `VERB -> NOUN` 78 |
| HMM + Viterbi | `PROPN -> NOUN` 375, `NOUN -> PROPN` 161, `VERB -> AUX` 105 |
| CRF | `PROPN -> NOUN` 210, `NOUN -> PROPN` 174, `VERB -> NOUN` 70 |
| Compact BiLSTM | `PROPN -> NOUN` 218, `NOUN -> PROPN` 158, `VERB -> NOUN` 59 |

Interpretation:

- Proper-noun vs common-noun boundaries are the hardest recurring distinction.
- HMM shows a more distinctive `VERB -> AUX` confusion, which fits its stronger dependence on transition statistics.
- The BiLSTM reduces several classical confusions, but it does not eliminate the `PROPN <-> NOUN` problem.

### 6.2 Per-tag behavior

Across the four runs, the easiest tags are highly regular categories such as:

- `PUNCT`
- `CCONJ`
- `DET`
- `PRON`
- `AUX`

The hardest tags are consistently:

- `X`
- `SCONJ`
- `PROPN`
- `SYM`
- `INTJ`

This pattern is stable across models. In other words, switching architectures helps overall quality, but it does not change which label types are intrinsically hardest.

### 6.3 Sentence length and OOV behavior

The model notebooks include inline bucket plots for:

- error rate by sentence length
- error rate by OOV bucket

The plots are consistent with the final ranking:

- `Compact BiLSTM` and `CRF` remain the most stable stronger models under harder conditions.
- `Averaged Perceptron` stays competitive but degrades slightly more.
- `HMM + Viterbi` shows the clearest drop on harder inputs.

Representative error examples in all notebooks also come disproportionately from:

- longer sentences
- higher-OOV sentences
- proper-name heavy contexts

### 6.4 Qualitative observations from representative errors

The saved inline examples show several recurring properties:

- the same difficult long sentences often reappear across models
- high-OOV cases are common among the hardest examples
- lexical ambiguity around names, nouns, and modifiers remains central

For example, the repeated difficult sentences from the `HumorUniversity`, `aggressivevoicedaily`, and `answers` subsets confirm that the hardest failures are not random single-token mistakes, but systematic context and OOV problems.

## 7. Practical Recommendation

### Best overall model by quality

Use `Compact BiLSTM` if the goal is maximum final accuracy.

Reason:

- highest test accuracy
- highest macro-F1
- strongest overall per-tag profile among the tested models

### Best practical trade-off

Use `CRF` if the goal is the best balance of quality, speed, and interpretability.

Reason:

- test quality is close to the neural model
- training is far cheaper than BiLSTM
- inference is much faster
- the model remains transparent and feature-based

### Best simple baseline

Use `Averaged Perceptron` if you want a simple, understandable, and still strong baseline.

Reason:

- nearly matches CRF in token accuracy
- straightforward to explain
- much simpler than the neural architecture

### Best pedagogical baseline

Use `HMM + Viterbi` when interpretability, tiny model size, and near-instant training matter more than final quality.

Reason:

- smallest model
- fastest training by a large margin
- easy probabilistic interpretation

## 8. Final Answer to the Research Question

In the current implementation, the best model by final quality is `Compact BiLSTM`. It reaches `0.952419` test accuracy and `0.904233` macro-F1, outperforming all classical baselines.

However, the best practical recommendation for a realistic compact course project is still `CRF`. Its test performance is close to the neural model at `0.946880` accuracy and `0.899161` macro-F1, while training and inference are much cheaper and the model is easier to interpret.

So the final answer is:

- if the only goal is the highest score, choose `Compact BiLSTM`
- if the goal is the strongest quality/speed/interpretability trade-off, choose `CRF`
