<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Genomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational genomics: sequence assembly, variant analysis and annotation.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-3%2C246-00284b" alt="3,246 skills">
  <img src="https://img.shields.io/badge/tools-867-00284b" alt="867 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-137-00284b" alt="137 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the genomics skill collection of AgenticScienceBuilder (ASB): 3,246 skills over 867 software tools and 2 composite workflows, distilled from 137 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 2,703 of the 3,246 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/genomics/v1` | ASB Genomics Skill Collection v1 | 3246 | 867 | 2026-11-14 | [10.5281/zenodo.24224625](https://doi.org/10.5281/zenodo.24224625) |

## Status

Collection `v1`, built from corpus cut `4b484d868384`, is released on `main` at tag `genomics-v1.1.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24224625](https://doi.org/10.5281/zenodo.24224625).

| Item | State |
|---|---|
| Release gate | `receipts/4b484d868384/gate_report.json` reports `warn`: 13 of 16 checks pass, 1 gives a warning and 2 are not applicable (`cut_manifest`, and `peer_review_claim` because the corpus holds no preprint or deposit DOI). The warning is on the scan of quote spans for personal data and dual-use content (`pii_dual_use`): 4 low-confidence findings of pattern `placeholder_subject`, flagged for human review. [`receipts/4b484d868384/VERIFY.md`](receipts/4b484d868384/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/genomics/v1` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/genomics/v1` |
| IRI resolution (2026-11-14) | Registered at w3id.org. It redirects (HTTP 302) to the collection landing page in the asb-skill-collections registry (HTTP 200), which names this repository and the Zenodo record. |
| DOI | [10.5281/zenodo.24224625](https://doi.org/10.5281/zenodo.24224625) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093084](https://doi.org/10.5281/zenodo.24093084) (all versions) |

## What is inside

| Item | Count | Path under `collections/genomics/v1/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 3,246 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 867 | `tools/` |
| Source papers | 137 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/4b484d868384/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `long-read-structural-variant` | long-read FASTQ data must be aligned, called by multiple structural-variant methods, verified per caller for read-depth support of copy-number events, merged, and compared with an external truth or high-confidence call set | 5 |
| `nanopore-5mcpg-detection` | nanopore POD5 or FAST5 signal must be basecalled with move tables, aligned while preserving move tags, analysed for 5mCpG with DeepMod2, aggregated with modkit, and checked against orthogonal methylation measurements | 5 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 2,639 |
| `noncommercial` | 490 |
| `restricted` | 0 |
| `unknown` | 117 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-genomics
claude plugin install genomics@asb-skills-genomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/genomics/v1/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-genomics/collections/genomics/v1/leaves/structural-variant-calling-from-long-reads <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-genomics
python3 collections/genomics/v1/bin/search_skills.py --query "structural variant calling from long reads" -k 3
python3 collections/genomics/v1/bin/search_skills.py --target workflows --query "structural variant calling from long reads" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. structural-variant-calling-from-long-reads
2. structural-variant-calling-long-reads
3. long-read-structural-variant-calling--dd2322b36136500f

1. long-read-structural-variant
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/4b484d868384/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `afa4911`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

136 of the 137 included papers are open access according to `corpus.yaml` (`access.is_oa`). The other paper is closed access and listed as `link-only`.

Each leaf names its source paper in `provenance.source_papers` (3,246 of 3,246 leaves). 2,703 of the 3,246 quote it, or its repository README, in their `## Evidence` section. In the other 543, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 532 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 9 the section holds a quote in a form the gate parser does not read. 2 have no `## Evidence` section.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- [`indicium-adapters-genomics`](https://github.com/HolobiomicsLab/indicium-adapters-genomics): the Indicium domain adapter for the same domain.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24224625](https://doi.org/10.5281/zenodo.24224625); its concept DOI, which covers all versions, is [10.5281/zenodo.24093084](https://doi.org/10.5281/zenodo.24093084).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
