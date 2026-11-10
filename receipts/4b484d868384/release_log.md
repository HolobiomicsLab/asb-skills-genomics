# Release log: genomics v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:11:15Z | 99c0626da751543a235010fbaa99ce7707007a57 | `{"candidate_tree": "sha256:c7553ab84da62ac105f3b44cd6e00df22293d400aad8472bf8ec4cebd29809f9", "gate_report": "c1bc1b33aa9dbdbae5ad1e3b53a84890c3c4770a7029fbdc8d718138ad44bf05"}` |
| deposit-plan | 2026-11-09T14:11:16Z | 99c0626da751543a235010fbaa99ce7707007a57 | `{"archive_sha256": "9b348995dcce47c4544da7748e8bd48d5935f7af1ea5664630ca8f84f3d1f912"}` |
| deposit-draft | 2026-11-09T21:05:43Z | 99c0626da751543a235010fbaa99ce7707007a57 | `{"archive_sha256": "9b348995dcce47c4544da7748e8bd48d5935f7af1ea5664630ca8f84f3d1f912"}` |
| record-doi | 2026-11-10T10:42:46Z | 99c0626da751543a235010fbaa99ce7707007a57 | `{"CITATION.cff": "71fb02edbe8a9649db299758224bb2bf163c713d49aa5408c98a28ed50afaafd", "README.md": "39634177d6284989e6deb2db55ddcd820d6f6f4588869d10fb79ce66a3c432e4", "RELEASE.yaml": "7b75e833f0e67bc8911643c417e0acbbce235b080f946e816b4fde4d81217a1a", "marketplace.json": "69971e3fc629c697fa3762b93405a8a4c0334c41bbc6a761a09294ef46a0e4eb", "plugin.json": "1890f13fa9d3146974cfa3070bc235af6940bcc791121d8ad8f4229acc734af3"}` |
| receipt | 2026-11-10T10:44:49Z | 99c0626da751543a235010fbaa99ce7707007a57 | `{"receipt": "05145737e697c391b1dca33e60e9d50288f249b4bc87cab546917a7641116199"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093085; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
