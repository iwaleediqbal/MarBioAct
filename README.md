# MarBioAct - Marine Bioactive Compound Library

![Compounds](https://img.shields.io/badge/compounds-83%2C052-0a7ea4)
![Format](https://img.shields.io/badge/format-SDF%20V2000%20(gzip)-2e8b57)
![Coordinates](https://img.shields.io/badge/coordinates-2D-lightgrey)
![RDKit](https://img.shields.io/badge/RDKit-2025.03.6-blue)
![Build](https://img.shields.io/badge/build-2026--09--23-555)
![Data licence](https://img.shields.io/badge/data%20licence-CC%20BY--NC--SA%204.0-orange)

MarBioAct is a structure library of **83,052 marine natural products** compiled from seven public natural product resources. Every compound is listed by at least one marine-specific collection (MarineNP, CMNPD or SWMD), and every record carries its source identifiers, organisms, marine evidence and a summary of the bioactivity data reported for it.

The library comes as two non-overlapping SDF files: compounds **with reported bioactivity** and compounds **without**. It is intended as a starting point for virtual screening, dereplication, chemical-space analysis and machine-learning data sets built on marine natural products.

---

## Files

```
MarBioAct/
├── README.md
├── LICENSE
└── data/
    ├── Marine_Bioactivity_Reported.sdf.gz
    └── Marine_Other_Compounds.sdf.gz
```

| File | Compounds | Size (gzip / unpacked) | Contents |
|---|---:|---|---|
| `data/Marine_Bioactivity_Reported.sdf.gz` | 26,390 | 25.3 / 129.8 MiB | Compounds with at least one imported assay record or qualitative activity annotation |
| `data/Marine_Other_Compounds.sdf.gz` | 56,662 | 46.9 / 269.4 MiB | Compounds without reported activity in the imported tables |
| **Total** | **83,052** | | |

The two files share no compound, so they can be concatenated into one library.

---

## Quick start

**Get the data**

```bash
git clone https://github.com/iwaleediqbal/MarBioAct.git
cd MarBioAct
```

A single file can also be downloaded directly, for example
`https://github.com/iwaleediqbal/MarBioAct/raw/main/data/Marine_Bioactivity_Reported.sdf.gz`.

**Decompress** (only needed for tools that cannot read `.gz` files)

```bash
gunzip -k data/*.sdf.gz
```

The unpacked files are standard V2000 SDF and open in Open Babel, Schrödinger Maestro, MOE, KNIME, DataWarrior and similar tools.

**Read in Python with RDKit** (no need to decompress)

```python
import gzip

from rdkit import Chem

path = "data/Marine_Bioactivity_Reported.sdf.gz"
with gzip.open(path, "rb") as fh:
    mols = [m for m in Chem.ForwardSDMolSupplier(fh) if m is not None]

print(len(mols))
print(mols[0].GetProp("DATABASE_ID"), mols[0].GetProp("GENERIC_NAME"))
```

**Load as a pandas table**

```python
from rdkit.Chem import PandasTools

df = PandasTools.LoadSDF("data/Marine_Bioactivity_Reported.sdf.gz")
exact = df[df["EXACT_CONCENTRATION_ROW_COUNT"].astype(int) > 0]
print(len(df), len(exact))
```

---

## What each record contains

Each SDF record holds a 2D structure and the data fields below. Fields with several values separate them with `; `. Two fields (`SOURCE_RECORD_IDS`, `SOURCE_TERMS`) are JSON objects keyed by source.

| Group | Fields |
|---|---|
| Identifiers | `DATABASE_ID` (MarBioAct ID), `DATABASE_NAME`, `GENERIC_NAME`, `SYNONYMS` |
| Structure | `SMILES`, `INCHI_IDENTIFIER`, `INCHI_KEY`, `FORMULA`, `MOLECULAR_WEIGHT`, `EXACT_MASS`, `COORDINATE_TYPE`, `DEPICTION_METHOD` |
| RDKit descriptors | `RDKIT_LOGP`, `RDKIT_POLAR_SURFACE_AREA`, `RDKIT_ACCEPTOR_COUNT`, `RDKIT_DONOR_COUNT`, `RDKIT_ROTATABLE_BOND_COUNT`, `RDKIT_FORMAL_CHARGE`, `RDKIT_HEAVY_ATOM_COUNT` |
| Provenance | `SOURCE_DATABASES`, `SOURCE_RECORD_IDS`, `SOURCE_ORGANISMS`, `SOURCE_TERMS` |
| Marine evidence | `MARINE_EVIDENCE`, `DIRECT_MARINE_COLLECTION` |
| Bioactivity summary | `BIOACTIVITY_STATUS`, `BIOACTIVITY_ROW_COUNT`, `EXACT_CONCENTRATION_ROW_COUNT`, `QUALITATIVE_ANNOTATION_COUNT`, `ACTIVITY_TYPES`\*, `ACTIVITY_TARGETS`\*, `QUALITATIVE_BIOACTIVITY`\*, `BIOACTIVITY_DETAILS` |
| Quality flags | `UNSPECIFIED_STEREO_COUNT`, `FRAGMENT_COUNT`, `SOURCE_IDENTITY_WARNINGS`, `IDENTIFIER_CORRECTION` |
| Notes | `USAGE_NOTE` |

\* Present only in `Marine_Bioactivity_Reported`, and only when the source data provide them.

`SYNONYMS`, `SOURCE_ORGANISMS`, `SOURCE_IDENTITY_WARNINGS` and `IDENTIFIER_CORRECTION` are left out of a record when they are empty. Molecular weight refers to the full exported form, including counter-ions.

Example (abridged):

```
> <DATABASE_ID>
MBC00036085641A94D546BE

> <GENERIC_NAME>
Bruceoside A

> <SOURCE_DATABASES>
MarineNP; NPASS

> <MARINE_EVIDENCE>
marine_collection; taxon_association

> <BIOACTIVITY_STATUS>
reported

> <ACTIVITY_TYPES>
EC50; ED50; IC50; INHIBITION
```

---

## Library composition

**Sources.** A compound is usually found in several sources, so the columns do not add up to the totals.

| Source | Bioactivity_Reported | Other_Compounds | Total | Share of library |
|---|---:|---:|---:|---:|
| MarineNP | 21,952 | 52,539 | 74,491 | 89.7% |
| NPASS | 22,902 | 48,916 | 71,818 | 86.5% |
| CMNPD | 15,982 | 15,551 | 31,533 | 38.0% |
| Natural Products Atlas | 5,893 | 10,159 | 16,052 | 19.3% |
| MIBiG | 700 | 369 | 1,069 | 1.3% |
| SWMD | 256 | 649 | 905 | 1.1% |
| GNPS | 0 | 2 | 2 | <0.1% |

**Marine evidence.** All 83,052 compounds carry `marine_collection` (listed by MarineNP, CMNPD or SWMD). In addition, 40,232 (48.4%) carry `taxon_association` (linked to a marine organism), 3,664 (4.4%) carry `context_candidate` (marine context in a source record) and 2 carry `spectral_collection`.

**Properties.**

| | Bioactivity_Reported | Other_Compounds |
|---|---:|---:|
| Median molecular weight, g/mol (IQR) | 407.30 (308.33–532.69) | 422.48 (316.44–591.71) |
| Molecular weight above 500 g/mol | 7,683 (29.1%) | 19,974 (35.3%) |
| Potentially unspecified stereochemistry | 5,885 (22.3%) | 18,899 (33.4%) |
| More than one molecular fragment | 697 (2.6%) | 909 (1.6%) |
| At least one exact concentration value | 8,571 (32.5%) | 0 |

No molecular weight or drug-likeness filter was applied.

---

## How the library was built

**Sources and versions**

| Resource | Version used |
|---|---|
| MarineNP | v1 CSV release (8 June 2025) |
| CMNPD | 1.0 public property and activity exports |
| Natural Products Atlas | v2024_09 TSV export |
| NPASS | 3.0 public files |
| SWMD | Public MOL archive |
| MIBiG | 4.0.1 (Zenodo) |
| GNPS | Faulkner Legacy spectral library |

---

## Read this before using the data

- **"Bioactivity reported" does not mean active.** The category includes negative results, toxicity data and qualitative observations. Only 8,571 compounds (32.5% of the file) have at least one exact concentration.
- **"Not reported" does not mean inactive.** It means that no activity was found in the imported tables.
- **Assay values are not in the SDF files.** Each record summarises its assay data (counts, endpoint types, targets). The `BIOACTIVITY_DETAILS` field refers to a companion assay table that is not part of this repository.
- **Endpoints are not comparable across assays.** An IC50 against an enzyme, an MIC against a bacterium and a cytotoxic IC50 in a cell line describe different experiments.
- **Marine evidence is not proof of marine isolation.** A listing in a marine collection or a link to a marine organism does not show that the analysed sample came from the sea. Many organism links come from lists that sources compiled from other databases, and several organisms (fungi, bacteria) also occur on land.
- **Not every entry is a natural product.** This build was not screened for synthetic drugs, reference compounds and industrial chemicals that some sources link to marine organisms through assay controls. Captopril and loratadine, for example, appear in `Marine_Bioactivity_Reported`. Review such entries before using the library as a natural-products-only set.
- **Structures are 2D.** Docking needs 3D conformers, protonation and tautomer states, and a decision on undefined stereocentres.
- **Overlap between sources is not independent support.** Databases often copy the same upstream records.
- **Coverage is limited to the files used.** MarinLit was not included because no authorised bulk export was available.
---

## Licence and source terms

The compiled library is released under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International licence (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/), as required by the share-alike condition of CMNPD. **Use is non-commercial only.**

The terms of each contributing source continue to apply. Every record lists its sources in `SOURCE_DATABASES` and their terms in `SOURCE_TERMS`.

| Source | Terms |
|---|---|
| MarineNP | CC BY 4.0 in the Zenodo record; the description states the data are free for non-commercial users |
| CMNPD | CC BY-NC-SA 4.0 |
| Natural Products Atlas | CC BY-NC 4.0 |
| MIBiG | CC BY 4.0 |
| GNPS | Spectra contributed directly to GNPS are CC0 by default |
| NPASS | No licence published |
| SWMD | No licence published |

This table summarises the terms found by the authors and is not legal advice.

---

## Citation

If you use MarBioAct, please cite the accompanying article (citation details will be added on publication):

> MarBioAct-DB: Integrating Marine Natural Product Structures and Bioactivity Evidence into a Traceable Compound Database. Manuscript in preparation, 2026.
