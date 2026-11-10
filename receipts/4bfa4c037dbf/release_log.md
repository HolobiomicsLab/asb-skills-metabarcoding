# Release log: metabarcoding v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:23:20Z | 1d80356212b434d5bc008faeaf6aa2c9a6602022 | `{"candidate_tree": "sha256:283b10379781e67cec7c2518d2c9e032484e752dd6d4320caca85d8635cf9ddc", "gate_report": "440ea48027e50cda609ea51b3be28a24b895dbfae199be0b4a8711e0a460a3a6"}` |
| deposit-plan | 2026-11-09T14:23:20Z | 1d80356212b434d5bc008faeaf6aa2c9a6602022 | `{"archive_sha256": "a7a557628f0757b3c35e40b2ec22fb485072192c52a440b1e93ecf4c68a48a4d"}` |
| deposit-draft | 2026-11-09T21:05:48Z | 1d80356212b434d5bc008faeaf6aa2c9a6602022 | `{"archive_sha256": "a7a557628f0757b3c35e40b2ec22fb485072192c52a440b1e93ecf4c68a48a4d"}` |
| record-doi | 2026-11-10T10:46:38Z | 1d80356212b434d5bc008faeaf6aa2c9a6602022 | `{"CITATION.cff": "d3640c817cca8ae7955fd47cb413c50c2d1b18293efd09ff536f34046eceac95", "README.md": "5013c76a9222d2fad173168d9845de02493043572d580f421e2dc45af510d353", "RELEASE.yaml": "5fd39c5758e25cd0b00a19bc510d2517373777d37fbd0802c6e84071cf1d6ee7", "marketplace.json": "29f6cd3d3b8172b6c97a80a17fc30ffdebbb5062236ab8320366e137fcbfac1e", "plugin.json": "9b9fa8de6b106a8a815ee20cb299b765f3265fc4d51fb895d64e0e54cce2ba7c"}` |
| receipt | 2026-11-10T10:47:08Z | 1d80356212b434d5bc008faeaf6aa2c9a6602022 | `{"receipt": "cfaed6e2f57113bd54fcce16d92842feb46654f58e4b9a6d3fe47061b86515c3"}` |
| manifest | 2026-11-10T10:51:07Z | cb8effa5f9e1a8e315fe5c81071b9e6ae63a7ec4 | `{"catalogue_jsonld": "90c2c2f440599118b681c0b47710df83cb04ef1a531dc98fa09ac1e11691ac1a", "collections_yaml": "e5f0fce1419650197e002e238b6c7297fae703fc9002960c89da0cf70096c513"}` |
| tag | 2026-11-10T11:06:51Z | cb8effa5f9e1a8e315fe5c81071b9e6ae63a7ec4 | `{}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093089; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
- OWNER: git tag metabarcoding-v1.0.0 cb8effa5f9e1a8e315fe5c81071b9e6ae63a7ec4
- OWNER: git push origin metabarcoding-v1.0.0
- OWNER: git tag metabarcoding-v1.0.0 cb8effa5f9e1a8e315fe5c81071b9e6ae63a7ec4
- OWNER: git push origin metabarcoding-v1.0.0
