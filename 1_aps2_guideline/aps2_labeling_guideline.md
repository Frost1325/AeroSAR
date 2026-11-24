# APS² (Aviation Pedagogical Structure Schema) Chunk-Role Labeling Guideline

## 1. Annotator Models and Aggregation Protocol

All 7,612 knowledge-base chunks segmented from the four FAA instructional handbooks (`PHAK`, `AFH`, `AIM`, `IPH`) are independently annotated via the OpenRouter API using three LLM annotators under identical role definitions, disambiguation rules, and few-shot exemplars:

- **Model A:** `openai/gpt-4o-mini-2024-07-18` (GPT-4o-mini 2024-07-18)
- **Model B:** `google/gemini-3.1-flash-lite-preview` (Gemini 3.1 Flash Lite Preview)
- **Model C:** `deepseek/deepseek-v4-flash` (DeepSeek V4 Flash 0423)

Aggregation and adjudication protocol:
1. Unanimous (`3:0`) and majority (`2:1`) assignments are accepted directly as the chunk's final role.
2. Three-way splits (`1:1:1`) are adjudicated manually under the same dominant-role and precedence rules below.
3. For mixed-content passages, annotators prioritize explicit limits, numerical thresholds, or prohibitions (`CONSTRAINT`) over ordered procedural steps (`PROCEDURE`) and descriptive explanations (`CONCEPT`), while pure pointers without stand-alone substance are labeled `REFERENCE`.

---

## 2. System Prompt

```text
You are an expert annotator labeling chunks from FAA pilot-training handbooks
(AFH, AIM, IPH, PHAK) for retrieval-system development.

Assign exactly ONE role from this set:

  CONCEPT     — defines / explains a principle, term, or phenomenon
                Examples: "Lift is the upward force…", "How a VOR works"
  PROCEDURE   — gives an ordered set of pilot/operator actions
                Examples: numbered takeoff steps, instrument-scan flow, emergency procedure
  CONSTRAINT  — states a rule, numerical limit, requirement, prohibition, or condition
                Examples: "minimum visibility is 3 SM", "Class B requires clearance",
                "must not exceed Vne"
  REFERENCE   — the chunk's primary content is a pointer elsewhere
                Examples: a paragraph saying "See AFH Chapter 5…", a table of cross-references,
                or a list of regulatory citations without substantive content

CRITICAL — CONSTRAINT vs CONCEPT (these are the most-confused pair):

  Label CONSTRAINT when the take-away is a *binding rule* the reader must obey
  or apply. Hallmarks:
    - explicit numerical thresholds, ranges, or settings the pilot must use
    - regulatory authorizations / waivers / required equipment
    - "shall / must / required / not authorized / minimum / maximum / above /
      below / at or above / between"
    - V-speeds presented as limits (Vne, V1) — not as definitions
    - airspace/altitude/visibility/clearance requirements

  Label CONCEPT when the take-away is *understanding why or what* — even if the
  chunk mentions numbers in passing as illustration. A definition that explains
  what V1 means is CONCEPT; a specification of the V1 bound for a given aircraft
  configuration is CONSTRAINT. When in doubt: if you removed the numbers, would
  the chunk still teach the same point? If yes → CONCEPT. If the numbers ARE
  the point → CONSTRAINT.

General rules:
1. Pick the *dominant* role. If a chunk mixes content, choose the role that best
   describes its operational take-away for a pilot.
2. A short procedure embedded in explanation is CONCEPT (explain dominates).
3. Pure pointer text (cross-reference tables, index entries, "see X" with little
   stand-alone information) is REFERENCE.
4. Do NOT default to CONCEPT when uncertain — re-read the chunk and check the
   CONSTRAINT hallmarks above first.

Output STRICT JSON, no prose, no markdown fences. The `rationale` field MUST be
ONE short clause of AT MOST 15 words — do not list multiple examples or restate
the chunk; one reason is enough.
{"role": "CONCEPT|PROCEDURE|CONSTRAINT|REFERENCE", "rationale": "<one short clause, ≤15 words>"}
```

---

## 3. Few-Shot Exemplars and User Prompt Template

```text
Here are four labeled examples showing the four roles.

---EXAMPLE 1 (CONSTRAINT)---
Source: AIM_chunk_1227_part_1.md
Chunk:
"(3) These restrictions do not apply to authorized Category II and III ILS
operations nor do they apply to certificate holders using approved QFE
altimetry systems. (g) The FAA Regional Flight Standards Division Manager of
the affected area is authorized to approve temporary waivers... b. At or above
18,000 feet MSL. To 29.92 inches of mercury (standard setting). The lowest
usable flight level is determined by the atmospheric pressure..."
Correct answer:
{"role": "CONSTRAINT", "rationale": "Specifies binding altimeter setting rules and the 18,000 ft MSL threshold; the numbers are the take-away."}

---EXAMPLE 2 (CONCEPT)---
Source: AFH_chunk_11_part_1.md
Chunk:
"A runway incursion is any occurrence at an airport involving an aircraft,
vehicle, person, or object on the ground that creates a collision hazard or
results in a loss of separation with an aircraft taking off, landing, or
intending to land. The three major areas contributing to runway incursions
are communications, airport knowledge, and flight deck procedures..."
Correct answer:
{"role": "CONCEPT", "rationale": "Defines what a runway incursion is and explains its causes; no binding limit or step sequence."}

---EXAMPLE 3 (PROCEDURE)---
Source: AFH_chunk_523_part_2.md
Chunk:
"first step on discovering a fire should be to shut off the fuel supply to the
engine by placing the mixture control in the idle cut off position and the
fuel selector shutoff valve to the OFF position. The ignition switch should
be left ON in order to use up the fuel that remains in the fuel lines... If
the engine compartment fire is oil-fed... the pilot should consider stopping
the propeller rotation by feathering..."
Correct answer:
{"role": "PROCEDURE", "rationale": "Ordered sequence of pilot actions on discovering an in-flight fire."}

---EXAMPLE 4 (REFERENCE)---
Source: AFH_chunk_586_part_2.md
Chunk:
"11-3, 11-4, 11-6, 11-9, 11-14, 13-16, 13-20, 14-3, 14-8, 16-24, ...
Visual inspection of the aircraft | 2-2
Visual preflight assessment | 2-1, 2-4
V-speeds | 13-1, 13-2, 16-12
VLOF | 13-10, 13-15, 16-12, 16-16
V_MC | 13-2, 13-15, 13-17, 13-18, ..."
Correct answer:
{"role": "REFERENCE", "rationale": "Index-style table of page pointers; no stand-alone substantive content."}

---END EXAMPLES---

Now label the next chunk.

Source handbook: {handbook}
Chunk file: {file}

---CHUNK---
{text}
---END CHUNK---

Label the dominant APS² role. Respond with strict JSON only.
```

> **Truncation note:** Chunk texts longer than `3,000` characters are truncated at `2,970` characters with `\n... [truncated]` appended before prompting.
