# Peptide LC Retention Time: Exploratory Analysis

An exploration of a peptide liquid chromatography dataset: 10,000 peptide sequences,
optional modifications, and measured retention times.

The brief noted that retention is already known publicly to be driven by hydrophobicity
and size, and that predictive models exist. I therefore did not set out to optimise a
predictive model. Instead, I explored what this dataset represents, how much it can be
trusted, where known chemistry appears, and where the simple picture breaks down.

## Running this

```bash
pip install -r requirements.txt
jupyter notebook portal_biotech_assessment.ipynb
```

The notebook expects:

```bash
./portal_ds_task/peptide_retention.csv
```

The data is not included, per the brief. The notebook runs top to bottom from a clean
kernel.
How I approached it
I followed two phases.

First, I established what the peptides and retention measurements represent, including
checking missing values, duplicates, distributions and whether the retention axis could
be interpreted directly.

Second, I investigated what drives retention and compared observations against known
peptide chemistry.

Throughout, I tried to test assumptions rather than import them. For example, I did not
assume that higher values represented "more hydrophobic" behaviour until the relationship
was supported by the dataset.

A recurring theme is that the analysis can identify consistent relationships, but cannot
always distinguish biological variation from measurement noise or sampling effects.

## 1. The retention time column is transformed, not raw

Retention time is a duration, so it cannot be negative. Approximately 19% of values are
below zero. This is not a small number of rows; it is a structural feature of
the column and indicates that these are not raw experimental retention times.

The distribution is approximately symmetric (skew ≈ 0.03, mean ≈ 44, median ≈ 44) and
centred around +44 rather than zero.

The interpretation used throughout is that this is a shifted or rescaled retention
axis. Negative values therefore represent earlier elution relative to a reference point,
rather than invalid measurements.

## 2. The peptides are biologically realistic
Amino acid usage follows expected proteomic patterns. Common Amino acids such as leucine
appear frequently, while rare Amino acids such as tryptophan occur less often. The dataset
does not resemble randomly generated sequences.

The data is also noticeably clean: few malformed sequences, few extreme values, and
well-behaved distributions. This suggests some upstream filtering or processing.

The ID column spans a much larger range than the number of rows, suggesting these 10,000
peptides are a subset of a larger collection. The sampling method is unknown.

## 3. No technical replicates, but matched chemical comparisons exist
I checked for identical peptide sequence and modification combinations appearing more than
once. None were found.

This means measurement noise cannot be calculated.

However, 63 peptide sequences appear more than once. Inspecting these showed that most
are the same peptide measured in different modified
and unmodified forms.

Using the matched peptide pairs identified above, I compared peptides with the same
sequence measured in an unmodified and oxidised state.

This controls for peptide-to-peptide differences in composition and length. A simple
comparison between all oxidised and all unmodified peptides would be confounded because
oxidised peptides are not randomly distributed across sequences.

The paired analysis shows a consistent reduction in retention after oxidation:

- mean shift ≈ -14 retention units
- median shift ≈ -13.5 units
- 95% confidence interval approximately [-16, -12]

I checked the robustness of this result using a parametric one-sample t-test as thr paired differences were approximately normally distributed.

## 4. Does Length affect RT?

Peptide length has a moderate correlation with retention (r ≈ 0.55), but the relationship is
not simply linear.

Longer peptides show a higher minimum retention, while the overall range remains broad.

Length is also confounded with composition because longer peptides contain more
opportunities for every residue.

## 5. Amino acid composition?
Amino acid fractions show a clearer relationship with retention.

Amino acids associated with higher retention include leucine, phenylalanine, isoleucine and
tryptophan. Amino Acids such as lysine, arginine and glutamate appear towards
lower retention.

The pattern is consistent with hydrophobicity affecting reversed-phase LC behaviour.

Does the order match known hydrophobicity?
The brief identifies hydrophobicity as an established driver of peptide retention, so I
compared the observed amino-acid ordering against a standard hydrophobic/hydrophilic
classification from BOC Sciences:

https://aapep.bocsci.com/resources/hydrophobic-and-hydrophilic-amino-acids.html

Hydrophobic Amino acids were coloured red and hydrophilic Amino acids blue.

The ordering mostly agrees:

Higher-retention Amino acids include leucine (L), phenylalanine (F), isoleucine (I),
tryptophan (W), valine (V), and methionine (M).
Lower-retention Amino acids include lysine (K), arginine (R), histidine (H), and
glutamate (E).
There are exceptions. Glycine and alanine are commonly classified as hydrophobic, but
their relationship with retention is weak or negative in this dataset.

This suggests that general hydrophobicity is related to retention but does not completely
capture chromatographic behaviour. Amino acids size, side-chain properties, and sequence
context likely also contribute.

## 6. Validating against published retention chemistry

To check that the patterns found in this dataset matched known peptide chemistry, I
compared the amino-acid effects learned from the data against the Meek (1980) HPLC
retention scale.

I fitted a Ridge regression model using amino-acid composition (the fraction of each
amino acid in a peptide) to estimate which residues were associated with higher or lower
retention. Ridge regression was used because amino-acid fractions are naturally
correlated: increasing one residue fraction affects the others.

I then compared the fitted amino-acid coefficients from this dataset with the published
Meek coefficients. The two rankings agree with a correlation of **r = 0.670**.

This means the model learned a similar pattern to an independent published retention
scale. In particular, hydrophobic residues such as leucine, isoleucine, phenylalanine and
tryptophan generally increase retention, while charged residues such as lysine and
arginine decrease it.

The differences are also useful:

- **Tryptophan** has a weaker effect here than in the published scale. It is the rarest
  amino acid in this dataset, so there are fewer examples to estimate its effect.

- **Histidine** behaves differently from the published scale. Its effect depends on
  chemical conditions such as pH, so this may reflect differences between experiments.

- **Proline** differs because this model only uses amino-acid composition. It ignores
  where residues occur in the sequence, and proline's effect can depend on position.

This comparison was not intended as a predictive model. It was a check that the patterns
found in this dataset agree with known peptide retention chemistry.

## What the data cannot tell me
* The retention transformation is unresolved, so effects are interpreted in relative
units rather than absolute time.

* Without technical replicates, measurement noise cannot be separated from biological
variation.

* The dataset is a subsample, so results may not generalise unchanged to the full
population.


* Correlation does not prove causation. The matched oxidation analysis provides the
strongest causal-style comparison, but still assumes oxidation is the main difference
between paired measurements.

## What I would do next
* Compare against an existing retention predictor such as DeepLC to understand the
transformation and unexplained residuals.

* Collect more examples of rare modifications to test whether their effects are
consistent.


## Reference
Meek, J.L. (1980). Prediction of peptide retention times in high-pressure liquid
chromatography on the basis of amino acid composition. Proceedings of the National
Academy of Sciences, 77(3), 1632–1636.

https://aapep.bocsci.com/resources/hydrophobic-and-hydrophilic-amino-acids.html