# POS Tagging Case Study

This repository is organized for submission in a notebook-first format.

## Main notebooks for Moodle submission

1. `notebooks/01_data_and_benchmark.ipynb`
   Data-focused notebook for loading the UD benchmark, checking split statistics, inspecting labels, and verifying the dataset before training.

2. `notebooks/02_averaged_perceptron.ipynb`
   Experiment notebook for the averaged perceptron baseline.

3. `notebooks/03_hmm_viterbi.ipynb`
   Experiment notebook for the HMM + Viterbi baseline.

4. `notebooks/04_crf.ipynb`
   Experiment notebook for the CRF model.

5. `notebooks/05_compact_bilstm.ipynb`
   Experiment notebook for the compact BiLSTM model.

## Repository structure

- `notebooks/`
  Primary experiment narrative and implementation used for the submitted case study.
- `data/`
  Raw benchmark data used by the notebooks.

## Dataset setup

The notebooks expect the raw **UD English EWT** files in `data/raw/`:

- `data/raw/en_ewt-ud-train.conllu`
- `data/raw/en_ewt-ud-dev.conllu`
- `data/raw/en_ewt-ud-test.conllu`

The repository does not bundle these raw files. Download them from the Universal Dependencies repository and place them into `data/raw/` before running the notebooks.

Dataset repository:

- `https://github.com/UniversalDependencies/UD_English-EWT`

Example:

```bash
mkdir -p data/raw
curl -L https://raw.githubusercontent.com/UniversalDependencies/UD_English-EWT/master/en_ewt-ud-train.conllu -o data/raw/en_ewt-ud-train.conllu
curl -L https://raw.githubusercontent.com/UniversalDependencies/UD_English-EWT/master/en_ewt-ud-dev.conllu -o data/raw/en_ewt-ud-dev.conllu
curl -L https://raw.githubusercontent.com/UniversalDependencies/UD_English-EWT/master/en_ewt-ud-test.conllu -o data/raw/en_ewt-ud-test.conllu
```

## Recommended review order

1. Open `notebooks/01_data_and_benchmark.ipynb`
2. Open the four model notebooks in order from `02` to `05`
3. Open `FINAL_REPORT.md` for the final written conclusions

## Notes

- The notebooks are intentionally structured, commented, and separated by purpose.
- The main implementation is intentionally embedded directly into the notebooks so the repository matches the expected notebook-centric submission format.
- Generated experiment artifacts are intended to appear in notebook cell outputs rather than be committed as separate result files.
