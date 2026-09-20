---
name: giab-high-confidence-truth-set-comparison
description: Use when evaluating the accuracy (precision/recall/FDR) of SNV calls
  from long-read SMS data on a human genome that has a GIAB high-confidence truth
  set available.
when_to_use_negative:
- The input reads are Illumina short reads — the comparison is designed for long-read
  SMS data (PacBio/ONT), where short-read callers like GATK HaplotypeCaller or FreeBayes
  are the appropriate benchmark instead
- The variant type of interest is indels or structural variants — Longshot calls only
  SNVs from pileup (indels can be genotyped only if supplied in an input VCF)
- The genome region lies outside the GIAB high-confidence BED — calls there cannot
  be scored as TP or FP without additional validation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Long-read SNV caller whose output VCFs are scored against the GIAB high-confidence
    truth set; installable via Bioconda (`conda install longshot`)
  repo: github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:d692b144805953163757c31efa783f4f08e13cd126cebd825b9ca14ab68873fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Longshot
  - Pacific Biosciences
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# giab-high-confidence-truth-set-comparison

## Summary

Benchmark long-read SNV callers (Longshot, WhatsHap, Clairvoyante) by restricting calls to Genome in a Bottle (GIAB) high-confidence regions of PacBio-sequenced human genomes and computing precision, recall, and FDR against the GIAB truth sets.

## When to use

Use when evaluating the accuracy (precision/recall/FDR) of SNV calls from long-read SMS data on a human genome that has a GIAB high-confidence truth set available.

## When NOT to use

- The input reads are Illumina short reads — the comparison is designed for long-read SMS data (PacBio/ONT), where short-read callers like GATK HaplotypeCaller or FreeBayes are the appropriate benchmark instead
- The variant type of interest is indels or structural variants — Longshot calls only SNVs from pileup (indels can be genotyped only if supplied in an input VCF)
- The genome region lies outside the GIAB high-confidence BED — calls there cannot be scored as TP or FP without additional validation

## Inputs

- NGMLR-aligned whole-genome PacBio SMS BAM/CRAM files for GIAB samples (NA12878, NA24385, NA24143, NA24149)
- Indexed hs37d5 (or hg38) reference FASTA
- GIAB high-confidence variant call sets and high-confidence region BED files
- Caller output VCFs from Longshot, WhatsHap, and Clairvoyante

## Outputs

- Per-sample precision, recall, and FDR values for each caller, restricted to GIAB high-confidence SNVs (e.g., table1_benchmark_results.csv)
- Per-caller wall-clock runtime measurements
- TP/FP/FN counts against the GIAB truth set

## How to apply

Run each caller on the NGMLR-aligned PacBio SMS BAMs (e.g., NA12878 at 44x, NA24385 at 62x, NA24143 at 27x, NA24149 at 23x aligned to hs37d5), then intersect every caller VCF with the GIAB high-confidence regions before counting true positives, false positives, and false negatives. For Longshot, apply the paper's filtering: remove variants with read depth > d + 5*sqrt(d) (d = median read depth), apply the density filter (>10 SNVs in a 500 bp window), and use a variable GQ cutoff matched to the median read coverage (a fixed GQ of 50 was used for short-read variants). Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and record wall-clock runtime per caller per sample. For WhatsHap, supply the Longshot candidate-variant VCF via the --potential_variants input so it performs genotyping rather than discovery; run Clairvoyante with its standard whole-genome settings.

## Related tools

- **Longshot** (Long-read SNV caller whose output VCFs are scored against the GIAB high-confidence truth set; installable via Bioconda (`conda install longshot`)) — github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf
```

## Evaluation signals

- Precision and recall are computed only over SNVs inside GIAB high-confidence regions; totals must reflect TP/(TP+FP) and TP/(TP+FN) after depth, density, and GQ filtering
- For NA12878 PacBio (44x), Longshot should reach precision ~0.995 and recall ~0.968, exceeding WhatsHap (0.972/0.975) and Clairvoyante (0.984/0.957), with FDR ~0.5%
- Longshot VCFs above a median-depth d must have variants with depth > d+5*sqrt(d) and dense clusters (>10 SNVs/500 bp) removed before scoring
- Runtime column populated for all three callers across the four GIAB samples, written to table1_benchmark_results.csv

## Limitations

- The GIAB and Platinum Genomes truth sets were generated using short-read datasets and are therefore biased in favor of short-read technologies; some Longshot false positives may actually be false negatives in the GIAB call sets
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, affecting calls in those regions
- FDR can be reduced further (to 0.3% on NA12878) by filtering known common indels, indicating residual indel-associated false SNVs; accurate short indel calling with SMS reads is challenging due to the high insertion/deletion error rate

## Evidence

- [discussion] GIAB truth sets used for accuracy assessment: "High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling"
- [discussion] GQ cutoff matched to median coverage for Longshot: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] Truth-set bias caveat: "some of the false-positive calls by Longshot may actually correspond to false negatives in the GIAB high-confidence call sets."
- [readme] Longshot scope and supported read types: "Longshot is a variant calling tool for diploid genomes using long error prone reads such as Pacific Biosciences (PacBio) SMRT"
