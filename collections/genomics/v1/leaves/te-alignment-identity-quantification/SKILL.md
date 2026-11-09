---
name: te-alignment-identity-quantification
description: Use when comparing polishing methods on repeat-rich TE regions of a HiFi
  assembly and you need per-element alignment identity against reference TE sequences
  to quantify correction versus overcorrection.
when_to_use_negative:
- Input TE sequences are already aligned and identity scores computed — re-running
  asm10 alignment adds no information
- The goal is read-to-reference mapping accuracy of noisy long reads rather than assembly-to-assembly
  consensus identity of finished TE sequences (use map-hifi/map-pb presets instead)
- Genome is assembled from noisy short reads without HiFi data; the underlying polishing
  methods and the identity gain expectations do not transfer
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3673
tools:
- name: minimap2
  role: Aligns polished TE sequences to Col-CEN TE references using the -cx asm10
    assembly-to-assembly preset for identity computation
  repo: https://github.com/lh3/minimap2/
- name: NextPolish2
  role: Produces the polished TE sequences whose identity gains/losses are quantified
    (run with -t 5 on sorted HiFi BAM plus yak k21/k31 datasets)
  repo: https://github.com/Nextomics/NextPolish2
- name: winnowmap
  role: Alternative HiFi mapper for preparing the sorted BAM input to polishing; weighted
    minimizers improve mapping within repetitive TE regions
  repo: https://github.com/marbl/Winnowmap
- name: yak
  role: Builds the k21/k31 k-mer dataset files from short reads that NextPolish2 uses
    to distinguish true alleles from errors
  repo: https://github.com/lh3/yak
- name: meryl
  role: Counts repetitive k-mers (k=15, distinct>0.9998) when winnowmap is used to
    prepare the HiFi mapping file
- name: samtools
  role: Sorts the HiFi alignment output into the sorted BAM consumed by NextPolish2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:56cc07b5ec369faa83b3161e0a983e5564e6680f03751f1d540079b01b513b86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NextPolish2
  - RepeatMasker
  - fastp
  - meryl
  - minimap2
  - samtools
  - winnowmap
  - yak
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# te-alignment-identity-quantification

## Summary

Quantifies transposable-element (TE) consensus accuracy after genome polishing by aligning polished TE sequences to curated TE references with minimap2 (-cx asm10) and computing per-TE identity ratios, enabling detection of both corrections and overcorrections in highly repetitive regions.

## When to use

Use when comparing polishing methods on repeat-rich TE regions of a HiFi assembly and you need per-element alignment identity against reference TE sequences to quantify correction versus overcorrection.

## When NOT to use

- Input TE sequences are already aligned and identity scores computed — re-running asm10 alignment adds no information
- The goal is read-to-reference mapping accuracy of noisy long reads rather than assembly-to-assembly consensus identity of finished TE sequences (use map-hifi/map-pb presets instead)
- Genome is assembled from noisy short reads without HiFi data; the underlying polishing methods and the identity gain expectations do not transfer

## Inputs

- Polished and unpolished assembly FASTA files containing the TE regions (e.g., NextPolish2 output asm.np2.fa and RM-polished output)
- Reference TE FASTA (158 Col-CEN centromeric TE sequences from TAIR; 149 subset successfully assembled)
- minimap2 executable with assembly-to-assembly support
- Simulated HiFi read alignment (sorted BAM) and yak k21/k31 k-mer datasets if polishing is performed in the same workflow

## Outputs

- Per-TE alignment identity ratios for each polishing method relative to Col-CEN references
- Proportion of TEs mapping at 100% identity before and after polishing
- Count/percentage of TEs losing identity per method (e.g., ~80.54% under RM)
- Summary table (te_identity_summary.csv)

## How to apply

Extract the TE loci of interest (e.g., the 149 successfully assembled Col-CEN centromeric TEs from a simulated A. thaliana hifiasm primary contig assembly) from both the unpolished assembly and the polished outputs of each method (e.g., NextPolish2 with -t 5, and the RM comparison pipeline). Align the extracted TE sequences against the reference TE set (Col-CEN TEs from TAIR) using minimap2 with the asm10 preset, which is tuned for closely related intra-species assembly-to-assembly alignments. Compute a per-TE alignment identity ratio (aligned identical bases / reference length or alignment extent) for each polishing method relative to the Col-CEN references, then compare against the pre-polishing baseline. Correct application is verified when the expected shifts are reproduced: NextPolish2 raises the fraction of TEs at 100% identity from 68.46% to 91.95% with no overcorrection, while RM drops it to 12.75% and lowers identity for ~80.54% of TEs. Save the per-TE and summary statistics (e.g., te_identity_summary.csv) so method-level conclusions are traceable to individual elements.

## Related tools

- **minimap2** (Aligns polished TE sequences to Col-CEN TE references using the -cx asm10 assembly-to-assembly preset for identity computation) — https://github.com/lh3/minimap2/
- **NextPolish2** (Produces the polished TE sequences whose identity gains/losses are quantified (run with -t 5 on sorted HiFi BAM plus yak k21/k31 datasets)) — https://github.com/Nextomics/NextPolish2
- **winnowmap** (Alternative HiFi mapper for preparing the sorted BAM input to polishing; weighted minimizers improve mapping within repetitive TE regions) — https://github.com/marbl/Winnowmap
- **yak** (Builds the k21/k31 k-mer dataset files from short reads that NextPolish2 uses to distinguish true alleles from errors) — https://github.com/lh3/yak
- **meryl** (Counts repetitive k-mers (k=15, distinct>0.9998) when winnowmap is used to prepare the HiFi mapping file)
- **samtools** (Sorts the HiFi alignment output into the sorted BAM consumed by NextPolish2)

## Examples

```
minimap2 -cx asm10 colcen_te_refs.fa polished_te.fa > te_asm10.paf
```

## Evaluation signals

- NextPolish2-polished TEs: proportion at 100% identity rises from 68.46% to 91.95% with no overcorrection (identity never falls below baseline for corrected elements)
- RM-polished TEs: proportion at 100% identity falls to 12.75%, with ~80.54% of TEs losing identity — a negative-control signature of overcorrection
- Per-TE identity ratios are computed for every one of the 149 assembled TEs; missing alignments should be investigated before summarizing
- Summary table te_identity_summary.csv contains per-method mean identity and per-TE deltas matching the reported figures
- minimap2 asm10 alignments exist for all polished-vs-reference TE pairs, confirming the asm10 preset (not a read-mapping preset) was used

## Limitations

- Identity ratios depend on the quality and completeness of the reference TE set; TEs absent from the reference cannot be scored
- NextPolish2's performance relies heavily on short reads to check whether a k-mer contains errors, so low short-read coverage or short-read biases can leave uncorrected errors
- Only TEs successfully assembled (149 of 158 Col-CEN centromeric TEs) can be evaluated; unassembled loci are excluded from the comparison
- Overcorrection and haplotype switch errors are the failure modes the identity comparison is designed to expose, particularly in tandem-repeat and centromeric regions

## Evidence

- [abstract] NextPolish2 can fix base errors in HiFi long-read assemblies without introducing excessive overcorrections and haplotype switch errors: "which can fix base errors remaining in those “highly accurate” genomes assembled from HiFi long reads without introducing excessive overcorrections and haplotype switch errors"
- [readme] asm10 preset for assembly-to-assembly alignment: [evidence span withheld — anchor status: external_doc]
- [methods] NextPolish2 invocation with threads, BAM, assembly and yak k-mer files: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [discussion] Performance relies on short reads to validate k-mers: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [intro] Overcorrection claim for repetitive regions: "NextPolish2 can fix base errors in “highly accurate” draft assemblies without introducing overcorrections, even in regions with highly repetitive elements"
