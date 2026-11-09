---
name: surface-receptor-isoform-annotation
description: Use when annotating and quantifying full-length complex isoforms at surface
  receptor gene loci from R2C2 ONT single-cell RNAseq, e.g. checking whether CD19,
  CD20 (Ms4a1), IGH, Ptprc, CD2, CD79b or CD37 express multiple isoforms across cells.
when_to_use_negative:
- Input is regular (non-R2C2) ONT reads — the Mandalorion README states it is 'not
  tested for use with regular ONT reads. I recommend using FLAIR instead for those.'
- Input is short-read Illumina data — the paper's Trinity comparison (task_008) shows
  short-read assembly largely failed to reproduce ONT-identified CD37 isoforms
- Target transcripts are under 500 bp, which were not captured due to stringent size
  selection during library preparation and the -I 500 isoform length filter
edam_operation: http://edamontology.org/operation_0264
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: Mandalorion
  role: 'Core pipeline: aligns R2C2 consensus reads to hg38 and defines high-confidence
    isoform consensus sequences at gene loci with the paper''s filter defaults'
  repo: https://github.com/christopher-vollmers/Mandalorion
- name: minimap2
  role: Splice-aware alignment of long cDNA reads to the genome, dependency configured
    in the Mandalorion config file
  repo: https://github.com/lh3/minimap2
- name: racon
  role: Consensus correction module, dependency configured in the Mandalorion config
    file
  repo: https://github.com/isovic/racon
- name: emtrey
  role: Converts minimap2 SAM output to PSL for downstream isoform processing, dependency
    in the config file
  repo: https://github.com/rvolden/emtrey
- name: C3POa
  role: Upstream R2C2 consensus generation that feeds R2C2_consensi.fasta and R2C2_subreads.fastq
    into Mandalorion
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/ncomms16027
    title: Nanopore long-read RNAseq reveals widespread transcriptional variation
      among the surface receptors of individual B cells
  source_content_hash: sha256:0a501db8322ff741850119bdffb6311e268453be0106c114569ec3e068b07f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christopher-vollmers/Mandalorion
  tools:
  - C3POa
  - Mandalorion
  - abPOA
  - emtrey
  - medaka
  - minimap2
  - racon
  edam_operation: http://edamontology.org/operation_0264
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# surface-receptor-isoform-annotation

## Summary

Identifies and quantifies full-length complex isoforms (alternative TSS/TES plus alternative splice sites) at B cell surface receptor loci by running the Mandalorion pipeline on R2C2 ONT reads from individual murine B1a cells, then testing differential isoform usage across cells. This reproduces the paper's finding that loci such as CD19, Ms4a1 (CD20), IGH, Ptprc (CD45), CD2, CD79b and CD37 express multiple complex isoforms.

## When to use

Use when annotating and quantifying full-length complex isoforms at surface receptor gene loci from R2C2 ONT single-cell RNAseq, e.g. checking whether CD19, CD20 (Ms4a1), IGH, Ptprc, CD2, CD79b or CD37 express multiple isoforms across cells.

## When NOT to use

- Input is regular (non-R2C2) ONT reads — the Mandalorion README states it is 'not tested for use with regular ONT reads. I recommend using FLAIR instead for those.'
- Input is short-read Illumina data — the paper's Trinity comparison (task_008) shows short-read assembly largely failed to reproduce ONT-identified CD37 isoforms
- Target transcripts are under 500 bp, which were not captured due to stringent size selection during library preparation and the -I 500 isoform length filter

## Inputs

- R2C2 consensus sequences (R2C2_consensi.fasta) from C3POa processing of ONT MinION reads
- R2C2 subreads (R2C2_subreads.fastq)
- Adaptor sequences (adapters.fasta)
- Mouse reference genome hg38 (hg38.fasta)
- GENCODE v29 annotation (gencodeV29.gtf)
- Mandalorion config file with paths to minimap2, racon, emtrey, blat, medaka and abPOA

## Outputs

- High-confidence isoform annotations and alignments per gene locus
- Tally of genes expressing complex isoforms (paper reports 169, including CD19, Ms4a1, IGH, Ptprc, CD2, CD79b, CD37)
- Genes with significant differential isoform usage across cells (paper reports 55; chi2 contingency test, alpha=0.001, Holm-Sidak correction)
- Counts of genes with alternative TSS/TES usage (696) and alternative splicing events (354)

## How to apply

Assemble a config file with paths to minimap2, racon, emtrey, blat, medaka and abPOA, then run 'python3 Mandalorion.py' with -c config_file, -p output path, -g gencodeV29.gtf, -G hg38.fasta, -a adapters.fasta, -f R2C2_consensi.fasta and -b R2C2_subreads.fastq so R2C2 consensus reads align to the hg38 genome and group into isoforms at gene loci. Apply the paper's read-support filters: -r 0.01 (minimum locus ratio), -R 5 (minimum reads per isoform), -n 2 (minimum feature count), -i 1 (minimum internal ratio), -I 500 (minimum isoform length) and -O 0,40,0,40 (overhang bounds). Define end sites with -u 10 and -d 50 buffers for TSS/polyA leniency, -w 1 splice site window, and discard isoforms whose A content within a 30 nt window around the polyA exceeds the -A 0.5 cutoff. Tally genes whose expressed isoforms contain both alternative TSS/TES and splice sites, confirming surface receptor loci CD19, Ms4a1, IGH, Ptprc, CD2, CD79b and CD37, then test differential isoform usage across the seven B1a cells with chi-squared contingency tests corrected by Holm-Sidak at alpha=0.001. Rationale: ONT long reads sequence full-length cDNA molecules, allowing complex isoforms to be resolved at the single-cell level where short-read approaches fail.

## Related tools

- **Mandalorion** (Core pipeline: aligns R2C2 consensus reads to hg38 and defines high-confidence isoform consensus sequences at gene loci with the paper's filter defaults) — https://github.com/christopher-vollmers/Mandalorion
- **minimap2** (Splice-aware alignment of long cDNA reads to the genome, dependency configured in the Mandalorion config file) — https://github.com/lh3/minimap2
- **racon** (Consensus correction module, dependency configured in the Mandalorion config file) — https://github.com/isovic/racon
- **emtrey** (Converts minimap2 SAM output to PSL for downstream isoform processing, dependency in the config file) — https://github.com/rvolden/emtrey
- **C3POa** (Upstream R2C2 consensus generation that feeds R2C2_consensi.fasta and R2C2_subreads.fastq into Mandalorion)

## Examples

```
python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq
```

## Evaluation signals

- Complex-isoform gene count should reproduce the reported 169 genes expressing isoforms with both alternative TSS/TES and splice sites
- The seven surface receptor loci CD19, Ms4a1 (CD20), IGH, Ptprc (CD45), CD2, CD79b and CD37 must be present as multiple complex isoforms in the output
- Differential isoform usage should recover approximately 55 significant genes using chi2 contingency tests with Holm-Sidak correction at alpha=0.001
- Reported isoforms should be >=500 nt (per the -I 500 minimum length and library size selection) and each supported by at least 5 reads (-R 5)
- No retained isoform should show A content above 0.5 within the 30 nt window around its polyA (-A 0.5 cutoff)

## Limitations

- Transcripts <500 bp were not captured due to stringent size selection during library preparation, so loci expressing only short isoforms are missed
- TSS/TES and splice sites are identified as ~20 bp bins rather than base-accurate positions due to ONT error rate
- Some alternative splicing events were not supported by single-cell Illumina data, possibly due to low per-cell sequencing depth and coverage
- Transcript quantification showed variation, which the authors suggest might be improved using UMIs during cDNA amplification
- Mandalorion is not tested with regular (non-R2C2) ONT reads; the README recommends FLAIR for those

## Evidence

- [methods] python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq: "python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq"
- [methods] -r  minimum ratio, proportion of reads that align to a locus required for an isoform (default 0.01): "-r  minimum ratio, proportion of reads that align to a locus required for an isoform (default 0.01)"
- [discussion] enabled us to identify 169 genes expressing complex isoforms containing both alternative TSS/TES and splice sites: "enabled us to identify 169 genes expressing complex isoforms containing both alternative TSS/TES and splice sites"
- [discussion] CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms: "CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms"
- [discussion] identify and quantify complex isoforms longer than ∼500 bp in an otherwise largely transcript length independent manner: "identify and quantify complex isoforms longer than ∼500 bp in an otherwise largely transcript length independent manner"
- [readme] Takes R2C2/C3POa and/or PacBio/ccs/lima data and defines high confidence isoform consensus sequences and alignments.: [evidence span withheld — anchor status: external_doc]
- [abstract] Our results show that we can identify and quantify complex isoforms at the single cell level: "Our results show that we can identify and quantify complex isoforms at the single cell level"
