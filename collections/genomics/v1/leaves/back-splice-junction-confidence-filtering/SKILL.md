---
name: back-splice-junction-confidence-filtering
description: Use when calling high-confidence BSJs and full-length circRNA isoforms
  from isoCirc/TRF consensus sequences mapped with minimap2, and you need to suppress
  false positives—especially BSJs involving novel splice sites—before aggregating
  per-tissue circRNA counts.
when_to_use_negative:
- Input is raw nanopore reads that have not yet undergone TRF consensus calling and
  minimap2 filtering (min consensus length 30, min copy number 2.0) — the filtering
  criteria operate on consensus alignments
- BSJs are intended to be called from short-read RNA-seq data; short reads do not
  determine full-length circRNA sequences or internal alternative splicing, so isoCirc's
  full-length confidence criteria do not apply
- Analysis targets subcellular-fraction RNAs or chromatin-associated RNAs — the article
  applied isoCirc only to whole cells and tissues
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: isoCirc
  role: Pipeline that implements consensus calling, consensus filtering/mapping, and
    high-confidence BSJ / full-length circRNA identification via its CLI flags (e.g.
    --cano-motif, --min-circ-dis, --rescue-low)
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2 (>= 2.11)
  role: Filters and maps consensus sequences used as the basis for BSJ confidence
    assessment
  repo: https://github.com/lh3/minimap2
- name: bedtools (>= v2.27.0)
  role: Dependency of the isoCirc pipeline used during annotation/interval handling
    of circRNA candidates
  repo: https://github.com/Xinglab/isoCirc
- name: Tandem Repeats Finder (TRF)
  role: Upstream consensus calling on tandem-repeat RCA reads that supplies the sequences
    to be confidence-filtered
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:ff116928b22b2d808aa0e8d6ec0a86b4167dd6d00fc36c9f54511d004fbb4288
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - Tandem Repeats Finder (TRF)
  - bedtools
  - bedtools (>= v2.27.0)
  - isoCirc
  - minimap2
  - minimap2 (>= 2.11)
  - pip
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# back-splice-junction-confidence-filtering

## Summary

Applies isoCirc's stringent criteria to remove false-positive back-splice junctions (BSJs) and full-length circRNA isoforms called from rolling-circle-amplified nanopore reads, using canonical GT/AG motif checks, mismatch/indel tolerances on the exonic sequence flanking the BSJ, and minimum genomic distance thresholds.

## When to use

Use when calling high-confidence BSJs and full-length circRNA isoforms from isoCirc/TRF consensus sequences mapped with minimap2, and you need to suppress false positives—especially BSJs involving novel splice sites—before aggregating per-tissue circRNA counts.

## When NOT to use

- Input is raw nanopore reads that have not yet undergone TRF consensus calling and minimap2 filtering (min consensus length 30, min copy number 2.0) — the filtering criteria operate on consensus alignments
- BSJs are intended to be called from short-read RNA-seq data; short reads do not determine full-length circRNA sequences or internal alternative splicing, so isoCirc's full-length confidence criteria do not apply
- Analysis targets subcellular-fraction RNAs or chromatin-associated RNAs — the article applied isoCirc only to whole cells and tissues

## Inputs

- TRF consensus sequences from isoCirc tandem-repeat RCA reads (long-read FASTA/FASTQ inputs)
- Reference genome FASTA
- Gene annotation GTF
- circRNA annotation BED/GTF
- minimap2 alignments of consensus sequences against the reference

## Outputs

- High-confidence BSJ calls with canonical GT/AG motif and flanking-sequence error checks
- Full-length circRNA isoform calls (35-column isocirc.out table with readCount column)
- isocirc.bed and isocirc_stats.out auxiliary outputs

## How to apply

After TRF consensus calling on tandem-repeat RCA reads (defaults: match 2, mismatch 7, indel 7, max period 2000, min score 100) and minimap2 filtering/mapping of consensus sequences (min consensus length 30, min copy number 2.0), identify high-confidence BSJs by requiring a canonical GT/AG splice motif, allowing at most 1 mismatch/indel/deletion within the 20-bp exonic sequence flanking the BSJ (0 within the 4-bp core), and enforcing a minimum genomic distance of 150 between back-splice sites. The filtering is deliberately much stricter for BSJs involving novel splice sites than for known splice sites, requiring near-perfect alignment between the isoCirc consensus sequence and the corresponding genomic sequence around the BSJ. The --rescue-low option can use high-mapping-quality reads to rescue low-mapping-quality reads when warranted. Filtered results are emitted as the 35-column isocirc.out table, whose readCount column feeds downstream per-tissue full-length isoform counting (e.g., at read >= 1).

## Related tools

- **isoCirc** (Pipeline that implements consensus calling, consensus filtering/mapping, and high-confidence BSJ / full-length circRNA identification via its CLI flags (e.g. --cano-motif, --min-circ-dis, --rescue-low)) — https://github.com/Xinglab/isoCirc
- **minimap2 (>= 2.11)** (Filters and maps consensus sequences used as the basis for BSJ confidence assessment) — https://github.com/lh3/minimap2
- **bedtools (>= v2.27.0)** (Dependency of the isoCirc pipeline used during annotation/interval handling of circRNA candidates) — https://github.com/Xinglab/isoCirc
- **Tandem Repeats Finder (TRF)** (Upstream consensus calling on tandem-repeat RCA reads that supplies the sequences to be confidence-filtered)

## Examples

```
isocirc -t 1 read.fa ref.fa anno.gtf circRNA.bed output --cano-motif GT/AG --min-circ-dis 150
```

## Evaluation signals

- Every accepted BSJ carries a canonical GT/AG splice motif (unless --cano-motif all is deliberately chosen)
- Flanking-region error budget respected: <=1 mismatch/indel/deletion in the 20-bp exonic flanking sequence, and 0 errors within the 4-bp core
- No accepted circRNA has a genomic distance between back-splice sites below 150
- Novel-splice-site BSJs show near-perfect consensus-to-genome alignment around the BSJ, per the stricter novel-site criteria
- Downstream reproduction check: per-tissue full-length isoform counts at read >= 1 match reported values (testis 115,348; prostate 18,312; skeletal muscle 19,261)

## Limitations

- isoCirc is far from saturation at current sequencing depth; more circRNA isoforms would be identified with deeper sequencing
- Filtering criteria are much stricter for BSJs involving novel splice sites than for known splice sites, so novel-site recovery is deliberately conservative
- The isocirc.com domain is a phishing site (per minimap2 README alert pattern; only trust GitHub-distributed tools)
- Full-length isoform counts vary enormously across tissues at comparable depth (e.g., testis 115,348 vs prostate 18,312), so thresholds tuned on one tissue may not transfer

## Evidence

- [methods] Identify high-confidence BSJs and full-length circRNAs: "Identifying high-confidence BSJs and full-length circRNAs:"
- [discussion] Stringent false-positive removal: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [discussion] Stricter criteria for novel splice sites: "the filtering criteria are much stricter (as compared to BSJs involving known splice sites)"
- [discussion] Near-perfect alignment requirement for novel-site BSJs: "requiring near-perfect alignment between the isoCirc consensus sequence and the corresponding genomic sequence around the BSJ"
- [methods] Consensus filtering and mapping with minimap2: "Filtering and mapping of consensus sequences (minimap2):"
- [methods] Rescue-low option semantics: "--rescue-low          Use high-mapping quality reads to rescue low-mapping quality reads"
- [readme] Canonical motif and distance flags in CLI: "[--cano-motif {GT/AG,all}] [--bsj-xid BSJ_XID] [--key-bsj-xid KEY_BSJ_XID] [--min-circ-dis MIN_CIRC_DIS]"
