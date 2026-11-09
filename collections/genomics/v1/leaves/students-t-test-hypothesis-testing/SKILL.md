---
name: students-t-test-hypothesis-testing
description: Use when per-read poly(A) tail lengths are already assigned to transcript
  isoforms (APA or splice isoforms) and you need to test, gene by gene, whether two
  or more isoforms have significantly different tail-length distributions.
when_to_use_negative:
- Input is raw PacBio subreads or untrimmed CCS reads — tail lengths must first be
  computed with the PAIso-seq pipeline (ccs, trim.py, run.pl, minimap2)
- Comparing tail lengths between different genes or between biological conditions
  rather than between isoforms of the same gene
- Tail-length data violate the QC criteria (pass number <10, tails <15 nt, >=20 non-A
  residues) — the published comparison was made only on reads passing these filters
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Computes per-isoform mean tail lengths and Student's t-tests between isoform
    groups; pipeline tested with R 3.5.1
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: minimap2 (v2.15-r905)
  role: Upstream splice-aware alignment (minimap2 -x splice) of trimmed CCS reads
    to mm10 that yields the isoform assignments used for grouping tail lengths
- name: run.pl
  role: PAIso-seq pipeline script that produces the per-read tail lengths and isoform/poly(A)-site
    calls fed into the t-tests
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:36d8e3541cab8e36ad912c5d033f9ffc300e7510b4128aa1388755a9cf66a1ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - R
  - minimap2 (v2.15-r905)
  - run.pl
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# students-t-test-hypothesis-testing

## Summary

Apply Student's t-tests in R to compare per-isoform poly(A) tail-length distributions (e.g. Ccnb1 APA1/APA2/APA3, Wee2 APA isoforms, Tcl1 splice isoforms) from PAIso-seq per-read tail measurements in mouse GV oocytes, testing whether isoforms of the same gene carry different tails.

## When to use

Use when per-read poly(A) tail lengths are already assigned to transcript isoforms (APA or splice isoforms) and you need to test, gene by gene, whether two or more isoforms have significantly different tail-length distributions.

## When NOT to use

- Input is raw PacBio subreads or untrimmed CCS reads — tail lengths must first be computed with the PAIso-seq pipeline (ccs, trim.py, run.pl, minimap2)
- Comparing tail lengths between different genes or between biological conditions rather than between isoforms of the same gene
- Tail-length data violate the QC criteria (pass number <10, tails <15 nt, >=20 non-A residues) — the published comparison was made only on reads passing these filters

## Inputs

- Per-read poly(A) tail lengths with APA/splice isoform assignments from the PAIso-seq pipeline (output of run.pl on trimmed CCS reads)
- QC-filtered read set: pass number >=10, clipped tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 in number
- Isoform grouping labels for target genes (Ccnb1 APA1/APA2/APA3, Wee2 APA isoforms, Tcl1 isoforms a/b/c/e)

## Outputs

- Per-isoform mean poly(A) tail-length summary table
- Pairwise Student's t-test p-values comparing tail-length distributions between isoforms of each gene (isoform_tail_length_summary.tsv)

## How to apply

Start from the per-read poly(A) tail-length table produced by the PAIso-seq pipeline (trim.py on CCS fasta, run.pl with a minimap2 -x splice mm10 index, and ccs.pass.txt pass counts), retaining only reads meeting the paper's QC: pass number >=10, clipped tail >=15 nt with >=5 continuous adenosines, and non-A residues <50% and <20 in number. Group tail lengths by isoform within each gene (Ccnb1 APA1/APA2/APA3, Wee2's two APA isoforms, Tcl1 splice isoforms a/b/c/e). In R (v3.5.1 in the published pipeline), compute per-isoform mean tail lengths and run Student's t-tests (var.equal=TRUE) pairwise between isoforms of the same gene, e.g. Ccnb1 APA3 vs APA1/APA2 or Wee2's two APA isoforms. Judge correctness by whether significant differences reproduce the reported directions: Ccnb1 APA3 longer than APA1/APA2 (p=0.0067), Wee2 APA isoforms differing (p=3.9e-12), Tcl1 splice isoforms differing (p=0.038, p=0.014), and write results to a per-isoform summary table with p-values (isoform_tail_length_summary.tsv).

## Related tools

- **R** (Computes per-isoform mean tail lengths and Student's t-tests between isoform groups; pipeline tested with R 3.5.1) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **minimap2 (v2.15-r905)** (Upstream splice-aware alignment (minimap2 -x splice) of trimmed CCS reads to mm10 that yields the isoform assignments used for grouping tail lengths)
- **run.pl** (PAIso-seq pipeline script that produces the per-read tail lengths and isoform/poly(A)-site calls fed into the t-tests) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
Rscript -e 'd <- read.delim("isoform_tails.tsv"); t.test(d$tail_len[d$isoform=="APA1"], d$tail_len[d$isoform=="APA3"], var.equal=TRUE)'
```

## Evaluation signals

- Wee2's two APA isoforms show a highly significant t-test difference (p=3.9e-12), the strongest signal in the analysis — a missing/different result indicates filtering or grouping errors
- Ccnb1 APA1 vs APA2 should be non-significant while Ccnb1 APA3 differs significantly (p=0.0067) and has a longer mean tail
- Tcl1 splice isoforms a/b/c/e yield significant pairwise differences (p=0.038 and p=0.014)
- All reads entering the tests satisfy the QC thresholds: pass number >=10, tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 in count
- Output table isoform_tail_length_summary.tsv contains one row per isoform with mean tail length and pairwise t-test p-values

## Limitations

- PacBio per-read cost limits coverage (~$5 per 1-K reads), so replicate correlation and t-test power depend on read depth per isoform
- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base, and extension-primer pairing can add a minimal number of extra A residues, slightly biasing tail lengths
- QC discards tails with <15 nt or >=20 non-A residues, so very short or heavily interrupted tails are excluded from the comparison

## Evidence

- [discussion] QC threshold for non-A residues: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [discussion] pass-number requirement for base-calling accuracy: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [readme] pipeline implemented in Perl and R: "This analysis pipeline is implemented in Perl and R."
- [readme] command producing per-read tail lengths and isoform calls: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
- [discussion] splice-aware alignment mode used for isoform assignment: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905)"
