# AeroSAR Derived Data Release

This package provides the non-code APS² annotation guideline, 7,612-chunk role annotations, and anti-leakage benchmark splits accompanying the paper *AeroSAR: Structure-Aware Retrieval-Augmented Generation for Instructional Aviation Manuals*.

All primary source texts come from four publicly available Federal Aviation Administration (FAA) handbooks (2023 PHAK `FAA-H-8083-25`, 2021 AFH `FAA-H-8083-3`, 2024 AIM, and 2017 IPH `FAA-H-8083-16`), and the base AeroQA benchmark is publicly hosted at <https://github.com/Frost1325/RADAR>.

---

## Directory Contents

```text
AeroSAR_derived_data/
├── README.md
├── 1_aps2_guideline/
│   └── aps2_labeling_guideline.md
├── 2_aps2_annotations/
│   └── aps2_chunk_roles.csv
└── 3_data_splits/
    └── aeroqa_anti_leakage_splits.json
```

---

## 1. `1_aps2_guideline/aps2_labeling_guideline.md`

Complete APS² annotation specification, including:
- Model snapshots via OpenRouter (`GPT-4o-mini 2024-07-18`, `Gemini 3.1 Flash Lite Preview`, `DeepSeek V4 Flash 0423`).
- Majority-vote and manual tie-adjudication policy.
- Verbatim system prompt, `CONSTRAINT` vs. `CONCEPT` disambiguation rules, four real-chunk few-shot examples, and JSON output schema.

---

## 2. `2_aps2_annotations/aps2_chunk_roles.csv`

Chunk-level APS² role annotations for all **7,612** FAA manual chunks.

- **Columns:** `file,handbook,role_a,role_b,role_c,final_role,vote_count,vote_pattern`
  - `file`: Chunk filename identifier (e.g., `AFH_chunk_100_part_1.md`).
  - `handbook`: Source manual (`AFH`, `AIM`, `IPH`, `PHAK`).
  - `role_a`, `role_b`, `role_c`: Independent role assignments from Model A, Model B, and Model C in `{CONCEPT, CONSTRAINT, PROCEDURE, REFERENCE}`.
  - `final_role`: Final adjudicated APS² role (`CONCEPT`: 3,893; `CONSTRAINT`: 1,479; `PROCEDURE`: 1,119; `REFERENCE`: 1,121).
  - `vote_count` / `vote_pattern`: Agreement pattern across the three models (`3` / `3-0`: 5,704; `2` / `2-1`: 1,791; `1` / `1-1-1`: 117).
- **Resolved truncated-output records:** In the raw Model B pass, two chunks (`AIM_chunk_1403_part_2.md` and `AIM_chunk_1469_part_16.md`) failed strict JSON parsing due to truncated rationales. For completeness of the released 7,612-row table, `AIM_chunk_1403_part_2.md` is resolved as `CONSTRAINT` (matching Models A and C and the chunk's binding 200 ft safety-distance rule, giving `3-0`) and `AIM_chunk_1469_part_16.md` is resolved as `CONCEPT` (recovering Model B's truncated `{"role": "CONCEPT", ...}` prefix and matching Model A, giving `2-1`). All inter-model agreement and $\kappa$ figures reported in Table 5 of the manuscript are computed directly over these 7,612 resolved rows (`Three-model overall = 82.78%, Fleiss' κ = 0.7422`; `A–B = 80.68%, κ = 0.7118`; `A–C = 82.78%, κ = 0.7483`; `B–C = 84.88%, κ = 0.7688`) with zero discrepancy.

---

## 3. `3_data_splits/aeroqa_anti_leakage_splits.json`

Anti-leakage partition mapping for all **2,588** AeroQA queries (`{"<qa_id>": "<split>"}`):
- `"train"`: **2,041** queries
- `"dev"`: **141** queries
- `"test"`: **165** queries
- `"excluded"`: **241** queries (development/test queries whose reference chunks intersect any training parent-chunk group, excluded to guarantee zero chunk-level or parent-group overlap)
