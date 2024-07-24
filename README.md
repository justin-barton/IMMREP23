# IMMREP23

Datasets from the IMMREP23 TCR specificity prediction challenge

![IMMREP23 Header](https://www.kaggle.com/competitions/54698/images/header)

## Contents
- [Competition Description](#competition-description)
- [Dataset Description](#dataset-description)
- [Files](#files)
- [Data Dictionary](#data-dictionary)
- [Data preparation methods](#data-preparation-methods)
- [Evaluation](#evaluation)
- [Citation](#citation)
- [References](#references)

## Competition Description

IMMREP23, the second annual IMMREP benchmark on TCR-epitope specificity prediction, ran from November 1, 2023 to December 11, 2023. Together with several experimental groups, we compiled a dataset of paired TCR data with annotated specificity to 21 pHLA (covering 6 distinct HLA molecules).

This challenge models TCR epitope recognition as a binary classification task. For a given test set of TCR-epitope pairs, the task of the model is to identify which pairs will bind and which will not bind.

Though the competition has ended, users can still join and make submissions on the Kaggle platform:
https://www.kaggle.com/competitions/tcr-specificity-prediction-challenge

Any new entries will not appear on the leaderboard, but the automatic scoring system will provide users with immediate feedback about the accuracy of their predictions.

## Dataset Description

Test (`test.csv`) and sample submission (`sample_submission.csv`) files are provided for competitors to complete with binding probability predictions in a `Prediction` column, indicating a probability between 0 and 1 that the peptide and TCR in a given row will bind.

**Note:** If there are peptides for which your model cannot make predictions, please use a default prediction of zero for these rows. A file of only zero-valued predictions will yield an AUC0.1 of 0.5.

Solutions (`solutions.csv`) provides the test set with labels and a `Usage` column which indicates whether the record is part of the public or private test set (for the public vs. private Kaggle leaderboards).

In addition to these, a sample training data file is provided (`VDJdb_paired_chain.csv`) with paired chain binding data compiled from VDJdb<sup>1</sup>. Note that this file only contains positive binding examples. As strategies vary with respect to how to generate synthetic non-binding examples, this is left as an exercise for competitors. As noted in the competition rules, competitors are not restricted to using this training set and may use any training data that is available to them.

## Files

- `data/VDJdb_paired_chain.csv`: A sample training dataset compiled from VDJdb<sup>1</sup>
- `data/test.csv`: The test set
- `data/sample_submission.csv`: A sample submission file in the correct format (`test.csv` + `Prediction` column)
- `data/solutions.csv`: The test set with labels (`test.csv` + `Label` and `Usage` columns)

## Data Dictionary

### Common columns

| Column | Description |
|--------|-------------|
| `Peptide` | The peptide/epitope |
| `HLA` | The HLA/MHC allele name |
| `Va` | The TCR alpha V gene name |
| `Ja` | The TCR alpha J gene name |
| `TCRa` | The TCR alpha amino acid sequence |
| `CDR1a` | The TCR alpha CDR1 amino acid sequence |
| `CDR2a` | The TCR alpha CDR2 amino acid sequence |
| `CDR3a` | The TCR alpha CDR3 amino acid sequence (positions 105-117) |
| `CDR3a_extended` | The TCR alpha CDR3 amino acid sequence (positions 104-118) |
| `Vb` | The TCR beta V gene name |
| `Jb` | The TCR beta J gene name |
| `TCRb` | The TCR beta amino acid sequence |
| `CDR1b` | The TCR beta CDR1 amino acid sequence |
| `CDR2b` | The TCR beta CDR2 amino acid sequence |
| `CDR3b` | The TCR beta CDR3 amino acid sequence (positions 105-117) |
| `CDR3b_extended` | The TCR beta CDR3 amino acid sequence (positions 104-118) |

### Train (VDJdb_paired_chain) specific columns

| Column | Description |
|--------|-------------|
| `Target` | The true binding label (1 for binder, 0 for non-binder) |

### Test/submission specific columns

| Column | Description |
|--------|-------------|
| `ID` | An identifier of the TCR-peptide pair used in test and submission files (NB: order matters, do not change the order or IDs) |
| `Prediction` | Your prediction of the probability of binding [0,1] |

### Solutions specific columns

| Column | Description |
|--------|-------------|
| `Label` | A binary flag indicating whether the TCR-peptide pair bind or not (1 or 0) |
| `Usage` | Indicates whether the TCR-peptide pair was used in the Kaggle public or private leaderboard scores ('Public' or 'Private') |

## Data preparation methods

To construct the full TCR sequences, the CDR3 sequences and V/J genes were submitted to the Thimble script in Stitchr<sup>2</sup> with the species set to human (e.g. "-s HUMAN"). In cases where multiple V- or J-gene alleles were listed for a given entry, all combinations were applied, and the set of TCRs recorded. In the current data, all such cases resulted in duplicated TCR, and a single entry was kept (randomly selecting the V/J gene from the multiple options).

TCRs were then submitted to ANARCI<sup>3</sup>, so that the individual CDRs could be annotated from the full sequence. Here, CDR1 was defined as positions 27-38, CDR2 as positions 56-65, CDR3 as positions 105-117, and CDR3_extended as positions 104-118 in the alignment.

### Negative data generation

Negatives for the test data were generated by swapping the TCRs from one peptide with TCRs binding to other peptides with a Levenshtein distance of more than 3. Here, 5 negatives were generated for each positive observation. This resulted in a positive to negative ratio of approximately 1:5.

## Evaluation

The evaluation metric for this competition was [Macro AUC0.1](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html). The ROC AUC score, commonly used in binary classification, is calculated as: 

$$ AUC = \int_0^1 TPR(FPR)\, dFPR $$

Where:
- ROC AUC represents the Area Under the ROC Curve.
- TPR stands for True Positive Rate, also known as Sensitivity or Recall.
- FPR stands for False Positive Rate, which is the complement of Specificity.
- dFPR represents the differential of False Positive Rate.

In our case, we are calculating the partial AUC up to a FPR of 0.1:

$$ AUC0.1 = \int_0^{0.1} TPR(FPR)\, dFPR $$

We calculate this AUC0.1 independently for each peptide in the test set and then calculate the arithmetic mean of these peptide-specific AUC0.1 scores to get the Macro AUC0.1.

$$ Macro\, AUC0.1 = \frac{1}{N} \sum_{1}^{N} AUC0.1(n) $$

Where N is the number of peptides in the test set. Note that since we are calculating a partial AUC, [McClish standardisation](https://doi.org/10.1177/0272989X8900900307) is used.

## Citation

Manuscript pending.

## References

1. Goncharov, M., Bagaev, D., Shcherbinin, D. et al. VDJdb in the pandemic era: a compendium of T cell receptors specific for SARS-CoV-2. Nat Methods 19, 1017–1019 (2022). https://doi.org/10.1038/s41592-022-01578-0

2. Heather, J., Spindler, M., Herrero Alonso, M., et al. Stitchr: stitching coding TCR nucleotide sequences from V/J/CDR3 information. Nucleic Acids Research, (2022). gkac190, https://doi.org/10.1093/nar/gkac190

3. Dunbar, J., Deane, C.M., ANARCI: antigen receptor numbering and receptor classification. Bioinformatics, Volume 32, Issue 2, January 2016, Pages 298–300, https://doi.org/10.1093/bioinformatics/btv552
