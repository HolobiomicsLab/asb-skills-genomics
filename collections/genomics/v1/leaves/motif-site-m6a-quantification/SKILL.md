---
name: motif-site-m6a-quantification
description: Use when you have nanopore per-read methylation predictions at known
  motif sites and need a single genome-wide percent m6A value per genome, or need
  to validate it against orthogonal global m6A measurements like ELISA.
when_to_use_negative:
- 'When quantifying methylation at untrained sequence contexts: mCaller training biases
  decrease accuracy at contexts dissimilar to the E. coli training examples, so motif-restricted
  aggregation should not be extended to unvalidated motifs'
- 'When the target methylation type is m5C rather than m6A: PacBio and this nanopore
  pipeline are reported as more suitable for m6A and m4C than m5C'
- When no known methyltransferase motif or positions file is available for the organism,
  as in the Dikarya fungi (C. neoformans, S. cerevisiae) where motifs could not be
  confidently assigned
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: mCaller
  role: Neural-network classifier that calls m6A per read from current deviations
    at motif positions, producing the per-read predictions aggregated here
  repo: https://github.com/al-mcintyre/mCaller
- name: make_bed.py
  role: mCaller summary script that collapses per-read predictions into the per-position
    methylation BED file aggregated across motif sites
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes reads and runs eventalign to produce the scaled-event TSV that mCaller
    consumes
  repo: https://github.com/jts/nanopolish
- name: scipy
  role: scipy.stats.spearmanr computes the rank correlation between nanopore-derived
    percent m6A and ELISA values
- name: matplotlib/seaborn
  role: Plots the ELISA-vs-nanopore scatter annotated with Spearman rho and p-value
- name: PacBio
  role: Orthogonal SMRT sequencing used to confirm the known motif sites at which
    methylation is quantified
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:751325afa1fb6f6b9740ed1ff1ffa18f2df8a66509b536b506aaa70561be18e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - PacBio
  - mCaller
  - make_bed.py
  - matplotlib
  - matplotlib/seaborn
  - nanopolish
  - pandas
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# motif-site-m6a-quantification

## Summary

Quantify genome-wide percent m6A by aggregating per-position methylation predictions (made only at known methyltransferase motif sites by mCaller) over all motif positions per species, then validate against orthogonal MethylFlash ELISA measurements via Spearman correlation.

## When to use

Use when you have nanopore per-read methylation predictions at known motif sites and need a single genome-wide percent m6A value per genome, or need to validate it against orthogonal global m6A measurements like ELISA.

## When NOT to use

- When quantifying methylation at untrained sequence contexts: mCaller training biases decrease accuracy at contexts dissimilar to the E. coli training examples, so motif-restricted aggregation should not be extended to unvalidated motifs
- When the target methylation type is m5C rather than m6A: PacBio and this nanopore pipeline are reported as more suitable for m6A and m4C than m5C
- When no known methyltransferase motif or positions file is available for the organism, as in the Dikarya fungi (C. neoformans, S. cerevisiae) where motifs could not be confidently assigned

## Inputs

- nanopolish eventalign TSV of scaled event values for ONT reads
- ONT FASTQ plus fast5 files indexed with nanopolish index
- Reference assembly FASTA per species
- Known motif positions file (positions.txt) or motif string (e.g. GATC) for mCaller
- mCaller per-read prediction file for make_bed.py
- Published MethylFlash ELISA global m6A values for the same species

## Outputs

- Per-position methylation summary BED file from make_bed.py
- One genome-wide percent m6A value per species (motif-site aggregate)
- Spearman rho and p-value versus ELISA values (expected rho = 0.97, p = 6.54E-6)
- ELISA-vs-nanopore percent m6A scatter plot (e.g. m6a_methylflash_spearman.png reproducing Fig 3b)

## How to apply

Run mCaller.py restricted to motif positions (e.g. -m GATC or -p positions.txt) with the trained model r95_twobase_model_NN_6_m6A.pkl, then run make_bed.py with a minimum read depth of -d 15 and a methylation threshold of -t 0.5 (a position counts as methylated when >= 50% of per-read observations are labelled methylated) to produce a per-position summary bed file. Aggregate the percent-methylated columns of that bed file over all motif positions per species to obtain one genome-wide percent m6A value per genome (eight microbial reference species from the ZymoBIOMICS community in the source paper). Restricting to known motif sites is the rationale: mCaller accuracy is high at trained sequence contexts (60-98% per-position accuracy) but unreliable at untrained contexts. Retrieve published MethylFlash ELISA global m6A values for the same species and compute the Spearman rank correlation with scipy.stats.spearmanr; the paper reports rho = 0.97 with p = 6.54E-6, which serves as the reproduction benchmark. Optionally plot the ELISA-vs-nanopore scatter with matplotlib/seaborn, annotating rho and the p-value to reproduce Fig 3b.

## Related tools

- **mCaller** (Neural-network classifier that calls m6A per read from current deviations at motif positions, producing the per-read predictions aggregated here) — https://github.com/al-mcintyre/mCaller
- **make_bed.py** (mCaller summary script that collapses per-read predictions into the per-position methylation BED file aggregated across motif sites) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes reads and runs eventalign to produce the scaled-event TSV that mCaller consumes) — https://github.com/jts/nanopolish
- **scipy** (scipy.stats.spearmanr computes the rank correlation between nanopore-derived percent m6A and ELISA values)
- **matplotlib/seaborn** (Plots the ELISA-vs-nanopore scatter annotated with Spearman rho and p-value)
- **PacBio** (Orthogonal SMRT sequencing used to confirm the known motif sites at which methylation is quantified)

## Examples

```
python make_bed.py -d 15 -t 0.5 -p <positions> -o species_methylation.bed; from scipy.stats import spearmanr; rho, p = spearmanr(nanopore_pct, elisa_pct)
```

## Evaluation signals

- Spearman correlation against MethylFlash ELISA values reaches rho = 0.97 with p = 6.54E-6 across the eight reference species
- make_bed.py was run with -d 15 (minimum read depth) and -t 0.5 (fraction of per-read observations labelled methylated); positions below these thresholds are excluded
- mCaller per-position accuracy falls in the reported 60-98% range at most motifs when checked against PacBio-confirmed truth sets
- Every aggregated percent m6A value derives only from positions at known, PacBio-confirmed motif sites; non-motif calls are treated as method-specific false positives
- The resulting scatter plot reproduces Fig 3b of the source paper when annotated with rho and p-value

## Limitations

- mCaller notably failed at one S. aureus motif, indicating incomplete motif coverage; accuracy is 60-98% at most motifs
- Training biases decrease accuracy at sequence contexts dissimilar to the E. coli training examples; models are R9.4 and R9.5 trained on E. coli data basecalled with Albacore
- Both single-molecule methods can fail to distinguish methylation types, as seen for the GATCGVNY motif in S. aureus
- Non-motif site calls are method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios, so they must be excluded from the aggregate

## Evidence

- [methods] make_bed.py minimum depth and threshold: "`-d` indicates the minimum read depth for inclusion"
- [methods] mod_threshold definition: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [discussion] mCaller accuracy range: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [intro] motif sites confirmed with PacBio: "using known motif sites confirmed with orthogonal PacBio data generated for ONT-sequenced E. coli DNA samples"
- [results] Spearman correlation of nanopore vs ELISA: [evidence span withheld — anchor status: not_found]
- [readme] mCaller usage from README: "usage: mCaller.py [-h] (-p POSITIONS | -m MOTIF) -r REFERENCE -e TSV -f FASTQ"
- [discussion] non-motif false positives: "suggesting that these are false positives that reflect different sources of noise for nanopore current deviations and PacBio IPD ratios"
