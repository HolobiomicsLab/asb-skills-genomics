---
name: gtf-transcript-feature-extraction
description: 'Use when a locus-resolved isoform inventory is needed: you have a genome-guided
  GTF annotation (e.g. galGal6 SSPO/SCO-spondin locus) and must enumerate exon counts
  per transcript and match them to CDS/protein sequences.'
when_to_use_negative:
- The transcriptome assembly has not yet been reconciled and annotated (raw StringTie
  GTF lacks coding/noncoding classification and reconciled gene IDs from annotate_my_genomes)
- The organism lacks an assembled genome and an NCBI GTF annotation, since the pipeline
  requires a reference genome as a guide
- The goal is de novo transcript quantification rather than structural feature (exon/CDS)
  extraction and isoform validation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: annotate_my_genomes
  role: Produces the reconciled, annotated final_annotated.gtf, cds.fa and prot.fa
    whose transcript records are filtered and parsed per locus
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: StringTie
  role: Generates the genome-guided transcript assembly (stringtie.gtf, settings -p
    1 -j 2 -c 2 -v -a 4) from merged PacBio+Illumina BAMs that becomes the GTF input
- name: AGAT
  role: GTF/GFF toolkit required by the pipeline for manipulating and validating the
    annotation records
  repo: https://github.com/NBISweden/AGAT
- name: GAWN
  role: Annotates proteins from transcripts via Blastx against Swissprot, contributing
    coding classification to the GTF records
  repo: https://github.com/enormandeau/gawn
- name: ncbi-blast+
  role: blastp (with -max_hsps 1 -max_target_seqs 1) validates extracted isoform proteins
    against the AJ866919 reference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:e066389c17a85bc442f1973c46ac1675fa71fac32c2b703025cdc9f7ab8ac226
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - GAWN
  - SAMtools
  - StringTie
  - annotate_my_genomes
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# GTF transcript feature extraction

## Summary

Parse exon/CDS records from an annotated genome-guided StringTie GTF (e.g. annotate_my_genomes' final_annotated.gtf) filtered to a target locus, count exons per transcript, and link transcripts to CDS/protein FASTA outputs for downstream isoform validation.

## When to use

Use when a locus-resolved isoform inventory is needed: you have a genome-guided GTF annotation (e.g. galGal6 SSPO/SCO-spondin locus) and must enumerate exon counts per transcript and match them to CDS/protein sequences.

## When NOT to use

- The transcriptome assembly has not yet been reconciled and annotated (raw StringTie GTF lacks coding/noncoding classification and reconciled gene IDs from annotate_my_genomes)
- The organism lacks an assembled genome and an NCBI GTF annotation, since the pipeline requires a reference genome as a guide
- The goal is de novo transcript quantification rather than structural feature (exon/CDS) extraction and isoform validation

## Inputs

- final_annotated.gtf — annotated, reconciled GTF from annotate_my_genomes (input stringtie.gtf reconciled against galGal6.gtf)
- galGal6 reference genome FASTA and NCBI GTF annotation (GCA_000002315.5)
- cds.fa — CDS sequences from the pipeline output
- prot.fa — deduced protein sequences from the pipeline output
- Reference protein sequence (e.g. AJ866919 deduced chicken SSPO protein)

## Outputs

- Locus-filtered GTF subset (e.g. SSPO transcript records with exon features)
- Per-transcript exon counts (expected isoform set: 106/106/105/17/25 exons)
- CDS/protein sequences per isoform with verified protein length (5270 aa for transcript 1)
- blastp identity report per isoform (98.88% to AJ866919)
- Validation table sspo_isoform_validation.csv with exon counts, protein lengths, and BLAST identity per isoform

## How to apply

Load the annotate_my_genomes output final_annotated.gtf (produced by running ./annotate-my-genomes -a stringtie.gtf -r galGal6.gtf -g galGal6.fa -c gawn_config.sh -t 20 -o output1) and filter transcript records to the locus of interest using the gene_id field and genomic coordinate overlap (e.g. the SSPO/SCO-spondin gene on the chicken galGal6 assembly). For each retained transcript, parse exon features from the GTF attribute field and count exons, confirming the expected isoform set (for SSPO: two 106-exon isoforms plus 105-, 17-, and 25-exon isoforms, five coding isoforms total). Extract the corresponding CDS and protein sequences from the pipeline outputs cds.fa and prot.fa, and verify the longest isoform's protein length (transcript 1 encodes 5270 aa). Finally, cross-validate protein identity by blastp against a reference protein (AJ866919) with -max_hsps 1 -max_target_seqs 1, expecting ~98.88% identity, and compile exon counts, protein lengths, and BLAST identity per isoform into a summary table such as sspo_isoform_validation.csv. Rationale: StringTie assemblies from merged PacBio+Illumina BAMs capture complex multi-exon loci (SSPO contains more than 105 exons) that reference annotations miss, so GTF-level feature extraction is the direct way to certify full-length recovery.

## Related tools

- **annotate_my_genomes** (Produces the reconciled, annotated final_annotated.gtf, cds.fa and prot.fa whose transcript records are filtered and parsed per locus) — https://github.com/cfarkas/annotate_my_genomes
- **StringTie** (Generates the genome-guided transcript assembly (stringtie.gtf, settings -p 1 -j 2 -c 2 -v -a 4) from merged PacBio+Illumina BAMs that becomes the GTF input)
- **AGAT** (GTF/GFF toolkit required by the pipeline for manipulating and validating the annotation records) — https://github.com/NBISweden/AGAT
- **GAWN** (Annotates proteins from transcripts via Blastx against Swissprot, contributing coding classification to the GTF records) — https://github.com/enormandeau/gawn
- **ncbi-blast+** (blastp (with -max_hsps 1 -max_target_seqs 1) validates extracted isoform proteins against the AJ866919 reference)

## Examples

```
awk '$0~/SSPO/ && $3=="exon" {match($0,/transcript_id "[^"]+"/); print substr($0,RSTART+14,RLENGTH-15)}' final_annotated.gtf | sort | uniq -c
```

## Evaluation signals

- The locus-filtered GTF yields exactly five coding SSPO isoforms with exon counts 106/106/105/17/25 (two 106-exon isoforms plus 105-, 17-, and 25-exon isoforms)
- All isoforms are classified protein-coding rather than lncRNA in the pipeline annotation
- Transcript 1's protein in prot.fa is exactly 5270 amino acids long
- blastp of the 5270-aa protein against AJ866919 with -max_hsps 1 -max_target_seqs 1 reports 98.88% identity
- sspo_isoform_validation.csv contains exon count, protein length, and BLAST identity for every isoform with no missing values

## Limitations

- Protein products of some assembled smaller SSPO isoforms remain unvalidated — the authors note 'the protein products from these transcripts remains to be validated'
- Smaller SSPO isoforms were only inferred as lower molecular weight bands (200 to 50 kD); isoform-protein correspondence is unconfirmed
- Pipeline (and thus this extraction workflow) is restricted to organisms with an assembled genome and an NCBI GTF annotation

## Evidence

- [abstract] correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons): "correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons)"
- [discussion] We input assembled transcripts in GTF format to the annotate_my_genomes pipeline, obtaining coding/noncoding annotations and reconciled GTF file: [evidence span withheld — anchor status: not_found]
- [methods] the AGAT toolkit must be installed: "the AGAT toolkit (https://github.com/NBISweden/AGAT) must be installed"
- [discussion] with the setting -max_hsps 1 -max_target_seqs 1 in blastp command: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command"
- [readme] Transcriptome annotation pipeline ... genome-guided transcriptome assemblies from StringTie: "annotate_my_genomes is a pipeline that aims to annotate genome-guided transcriptome assemblies from StringTie"
