---
name: kmer-frequency-counting
description: Use when you need k-mer frequency profiles from raw Illumina reads or
  genome assemblies — e.g. GenomeScope genome-size fitting, filtering repetitive fragments
  by 25-mer copy number, or comparing 6-mer composition between assemblies.
when_to_use_negative:
- You need per-base consensus accuracy or polishing of nanopore assemblies — use Racon/Pilon
  (or the ONT assembly-polish pipeline) instead, since k-mer frequencies only capture
  composition bias
- You need to distinguish haplotypes or map structural variants — k-mer copy-number
  statistics conflate up to four haplotypes in the hybrid eel assembly
- The reads are variable-length with quality decay — truncate/trim first (e.g. 76
  nt, Sickle) so k-mer quality is uniform before counting
edam_operation: http://edamontology.org/operation_0232
edam_topics: []
tools:
- name: Jellyfish
  role: Counts k-mer frequencies (k=6, 25, 19–25) on reads, merged fragments, and
    assembly FASTAs to produce histograms for GenomeScope fitting, seed filtering,
    and 6-mer correlation analysis
- name: GenomeScope
  role: Fits the Jellyfish k-mer histograms to estimate haploid genome size, repetitive
    fraction, and heterozygosity
- name: FLASh
  role: Merges paired-end Illumina reads (min overlap 15 nt) into fragments prior
    to 25-mer counting for seed selection
- name: Sickle
  role: Quality-trims FASTQ reads so k-mer counting operates on uniform-quality sequence
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:0f8a6c35d433055dfa7a724360fb244fb67b4a06073596892379094c1ac1823e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - FLASh
  - GenomeScope
  - Jellyfish
  - Sickle
  edam_operation: http://edamontology.org/operation_0232
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# kmer-frequency-counting

## Summary

Count and normalize k-mer frequencies with Jellyfish to support genome-size estimation (~860 Mbp eel genome), 'not too repetitive' seed selection, and cross-assembly composition comparison in the European eel TULIP pipeline.

## When to use

Use when you need k-mer frequency profiles from raw Illumina reads or genome assemblies — e.g. GenomeScope genome-size fitting, filtering repetitive fragments by 25-mer copy number, or comparing 6-mer composition between assemblies.

## When NOT to use

- You need per-base consensus accuracy or polishing of nanopore assemblies — use Racon/Pilon (or the ONT assembly-polish pipeline) instead, since k-mer frequencies only capture composition bias
- You need to distinguish haplotypes or map structural variants — k-mer copy-number statistics conflate up to four haplotypes in the hybrid eel assembly
- The reads are variable-length with quality decay — truncate/trim first (e.g. 76 nt, Sickle) so k-mer quality is uniform before counting

## Inputs

- Raw Illumina paired-end reads (e.g. SRR5235521), truncated to 76 nt
- FLASh-merged paired-end Illumina fragments (min overlap 15 nt)
- Genome assembly FASTA files: AZBK01 draft, TULIP scaffolds, Racon-corrected scaffolds, Pilon-polished scaffolds
- Seed FASTA files (e.g. eel_seeds_285.fasta) with unique numeric identifiers

## Outputs

- Jellyfish k-mer count histograms (one per k and per dataset)
- GenomeScope fit summary (haploid genome size 854.0–866.5 Mbp, repetitive fraction 15.5–20.0%, heterozygous fraction 1.48–1.59%)
- Filtered seed sets: 5,019,778 x 270 bp and 873,058 x 285 bp FASTA sequences
- 6-mer relative-frequency profiles, 4x4 Pearson correlation matrix (CSV) and heatmap (PNG)

## How to apply

Choose k according to the goal: k=19–25 on reads truncated to a uniform 76 nt feed GenomeScope histograms for haploid genome size, repeat fraction, and heterozygosity estimation; k=25 counts on FLASh-merged fragments flag any fragment containing a 25-mer occurring more than 25 times for removal under the 'not too repetitive' seed criterion; k=6 on FASTA assemblies (AZBK01 draft, TULIP, Racon, Pilon) yields canonical 6-mer histograms normalized to relative frequencies for Pearson correlation heatmaps. Count with Jellyfish in canonicalized mode and export count histograms per dataset. Decision points: select the GenomeScope fit whose haploid size lands in the reported 854.0–866.5 Mbp range, and size-select remaining seed fragments to exactly 270 bp (5,019,778 seeds, ~1.58x coverage) or 285 bp (873,058 seeds, ~0.29x coverage) with unique numeric identifiers (excluding 0). Validation relies on histogram presence, sequence-count/length checks, and reproducible correlation patterns.

## Related tools

- **Jellyfish** (Counts k-mer frequencies (k=6, 25, 19–25) on reads, merged fragments, and assembly FASTAs to produce histograms for GenomeScope fitting, seed filtering, and 6-mer correlation analysis)
- **GenomeScope** (Fits the Jellyfish k-mer histograms to estimate haploid genome size, repetitive fraction, and heterozygosity)
- **FLASh** (Merges paired-end Illumina reads (min overlap 15 nt) into fragments prior to 25-mer counting for seed selection)
- **Sickle** (Quality-trims FASTQ reads so k-mer counting operates on uniform-quality sequence) — https://github.com/najoshi/sickle

## Examples

```
jellyfish count -m 25 -s 500M -t 8 -C merged.fasta && jellyfish histo merged.jf > merged_25mer.histo
```

## Evaluation signals

- GenomeScope haploid genome size falls within 854.0–866.5 Mbp, consistent with the k-mer-analysis genome size of ~860 Mbp reported for A. anguilla
- Seed files contain exactly 5,019,778 sequences of 270 bp and 873,058 of 285 bp, matching the reported 1.58x and 0.29x coverage, with unique numeric identifiers (excluding 0) and uniform sequence lengths
- genomescope_summary.txt exists and reports repetitive fraction 15.5–20.0% and heterozygous fraction 1.48–1.59%
- The 4x4 6-mer Pearson correlation matrix reproduces the qualitative pattern: near-unity correlation after Pilon correction, reduced correlation for uncorrected TULIP/Racon (homopolymer underrepresentation), and slight high-CG 6-mer underrepresentation in the draft/Pilon assemblies
- All discarded seed fragments contain at least one 25-mer with count > 25 (repetitive-sequence criterion)

## Limitations

- Repetitive filtering by 25-mer copy number means large repetitive regions and (sub)telomeric repeats may lack frequent unique 285 bp islands and could be absent from the TULIP assembly
- K-mer frequency comparison detects platform bias (homopolymer underrepresentation in nanopore assemblies; slight high-CG underrepresentation in Illumina draft/Pilon) but cannot establish which assembly is correct without a high-quality reference
- Inflated short-read assembly sizes versus missing genomic fractions cannot be distinguished by k-mer statistics alone
- TULIP seeds must contain no 25-mer occurring more than 25 times; repetitive seeds are allowed in principle but make graph untangling much slower

## Evidence

- [discussion] K-mer based genome size estimation anchored to the reported eel genome size: "the genome size predicted from k-mer analysis (~860 Mbp)"
- [readme] Seed selection criterion applied via 25-mer copy-number filtering in TULIP input preparation: "These are pre-selected seed sequenced, with selection criteria 'not too repetitive'. Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph"
- [abstract] Genome size context for the k-mer counting target organism and computational scale: "For this 860 Mbp genome, the entire computational process takes two days on a single CPU."
