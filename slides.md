---
title: Referenceless Contig Stitcher (MiCall)
theme: seriph
background: https://raw.githubusercontent.com/Donaim/proviral-pres1/refs/heads/master/assets/forest.webp
info: |
  ## MiCall Referenceless Contig Stitcher
  Increasing usable assemblies in denovo mode
class: text-center
drawings:
  persist: true
mdc: true
zoom: 0.85
hideInToc: true
---
<br><br>
<h1 style='color:#20241f;opacity:85%;text-shadow:2px 2px 14px white'><b>Referenceless Contig Stitcher</b></h1>
<h3 style='color:white;opacity:80%;text-shadow:1px 1px 0px black,-1px 1px 0px black,1px -1px 0px black,-1px -1px 0px black'>Safe overlap stitching to rescue fragmented denovo assemblies</h3>
<div class="absolute bottom-5 left-3" style='opacity:85%;text-shadow:1px 1px 0px black,-1px -1px 0px black'>
  <b>23 Sep 2025 · Internal Tech Update</b>
</div>
<!-- Notes: Internal module; goal = increase usable-yield in denovo mode. -->

---

## 1. Title & One‑Liner

Referenceless Contig Stitcher: increase usable assemblies in denovo mode with zero workflow changes.

Sub: Reference-free overlap stitching that abstains unless confident.

<!-- Keep concise; pipeline-internal framing. -->

---
## 2. Current State (The Pain)

- Denovo samples fragment into multiple contigs.
- Fragmented cases lacking a single near-full-length contig are often discarded or deferred.
- Lost downstream analyses; re-runs delay insight.
- No reference (by design) here → reference-aware stitchers can’t help yet.

<img src="./assets/progress.webp" alt="workflow" class="w-80" />

<!-- Focus on usable-yield loss; avoid manual merge claims. -->

---
## 3. What It Does (Input → Process → Output)

Input (denovo mode): assembler FASTA (IVA, etc.)

Process:
 - Reference-free overlap scoring
 - High-confidence stitching only (≈99 effective matches threshold)
 - Event logs (DeterminedOverlap, CombinedContigs, Covered)
 - Abstains on ambiguity or low score

Output:
- Stitched FASTA OR original contigs (unchanged) if no safe join

<!-- Safety via abstention, not aggressive merging. -->

---
## 4. Where It Fits (Integration in MiCall)

- Invoked inside MiCall `sample.py` (Kive) only in denovo mode
- No user action; not exposed as separate tool
- Reference-aware stitchers still run later (complementary)
- Non-denovo / clinical flows unaffected

<img src="./assets/miseq.webp" alt="integration" class="w-70" />

---
## 5. Why It’s Safe (Guardrails)

- Conservative threshold (~99 matches) for merges
- Fully covered fragments discarded (prevents duplication)
- Alternative path pool bounded (SortedRing ≤ 999)
- Detailed event logging for each decision
- Abstain strategy: uncertain overlaps → keep originals

<img src="./assets/pipelines.webp" alt="guardrails" class="w-70" />

---
## 6. How It Works (Condensed Rules)

Principles: Scale-Dependent Credibility | Length Prioritization | Ambiguity Omission

Rules:
1. Merge non-overlapping contigs (strip ambiguous ends)
2. Merge overlapping contigs at concordance-optimal cut point (local alignment + sliding concordance)
3. Split contigs at large covered gaps (>21 nt)
4. Discard contigs fully covered by others

---
## 7. Outcomes (Metrics to Collect)

User experience: unchanged (transparent)

Validation metrics (denovo set):
- Usable-yield (% single/target contig samples) ↑
- Contigs per sample (median / p95) ↓
- N50 & total assembled length ↑ or stable
- Event log ratios (merges / abstentions) sanity
- Runtime overhead (target < Y%)

---
## 8. Validation & Rollout Plan

Phase 0 (now): finalize metric definitions & golden dataset
Phase 1 (wks 1–2): run on validation denovo set, capture metrics
Phase 2 (wks 3–4): compare before/after, review logs, sanity checks
Phase 3 (wks 5–6): include in MiCall release (denovo only) + release notes

Success (draft):
- ≥ X% relative increase usable-yield
- No increase in suspected chimeras
- Runtime overhead < Y%

---
## 9. FAQ (Selected)

Q: Affect non-denovo?  A: No
Q: Replace existing stitchers?  A: Complements later reference-aware step
Q: Bad joins risk?  A: Conservative scoring + abstain path
Q: Performance?  A: Vectorized scoring; bounded alternatives
Q: Logging?  A: Detailed event log for reproducibility

---
## 10. Decision

Approve inclusion in next MiCall release (denovo mode only)?

Success = higher usable-yield, stable quality, minimal runtime cost.
Fallback = disable flag; no external deps added.

---
## Appendix A – Rule Flow (Conceptual)

```mermaid
flowchart TD
  A[Input Contigs] --> B{Overlap?}
  B -- No --> C[Rule 1: Concatenate (strip ambiguous ends)]
  B -- Yes --> D[Rule 2: Concordance Cut]
  D --> E{Big Gap?}
  C --> E
  E -- Yes --> F[Rule 3: Split @ midpoint (>21 nt covered)]
  E -- No --> G{Fully Covered?}
  F --> G
  G -- Yes --> H[Rule 4: Discard covered]
  G -- No --> I[Candidate Path]
  H --> I
  I --> J[Most Probable Path Selection]
  J --> K[Output Contigs]
```

---
## Appendix B – Core Stitch Entry (Excerpt)

```python
from micall.utils.referenceless_contig_stitcher import (
  referenceless_contig_stitcher, read_contigs, write_contigs
)

def referenceless_contig_stitcher_with_ctx(input_fasta, output_fasta):
  contigs = tuple(read_contigs(input_fasta))
  log(events.Loaded(len(contigs)))
  if output_fasta is not None:
    contigs = tuple(stitch_consensus(contigs))  # overlap consensus + O2 pass
    log(events.Outputting(len(contigs)))
    write_contigs(output_fasta, contigs)
  return len(contigs)
```

Notes:
- `stitch_consensus` = overlap stitching + greedy O2 merge.
- Logging emits structured events (see next slide) for reproducibility.

---
## Appendix C – Sample Log Events

```text
Loaded 5 contigs.
Initializing initial seeds...
Starting with 5 initial seeds.
Constructed a path of length 4120.
Calculated cutoff for an overlap of size 132 between contigs c1 and c3 to be (54, 78).
Overlap between contigs c1 and c3 has aligned size 132, 129 matches, and the score of 0.977.
Found a significant overlap of size 132 between contigs c1 and c3, resulting in contig c1_c3.
Contig c1_c3 covers contig c2 completely.
Initial overlap stitching produced 2 contigs.
Giving up on attempts to stitch more overlaps since most probable is a singleton.
Outputting 2 contigs.
```

---
## Appendix D – Metrics Template (Fill Later)

| Metric | Baseline | After | Target | Status |
|--------|----------|-------|--------|--------|
| Usable-yield (% single contig) |  |  | +X% rel | Pending |
| Median contigs/sample |  |  | ↓ | Pending |
| N50 (bp) |  |  | ≥ | Pending |
| Runtime overhead (%) |  |  | < Y | Pending |
| Suspicious chimeras (controls) |  |  | = | Pending |
| Merge abstention rate |  |  | Reasonable band | Pending |

Definition notes:
- Usable-yield = % samples with 1 (or target threshold) contig after stitcher.
- Abstention rate outside bounds triggers review of MIN_MATCHES threshold.

---
## Appendix E – Future Enhancements (Optional)

- Adaptive thresholding (dynamic based on length / entropy)
- Parallel scoring across contigs (multiprocessing pool)
- Event log to JSON for automated QA dashboards
- Validation harness auto-report (CI artifact)

---
layout: cover
background: https://cover.sli.dev
zoom: 2.0
hideInToc: true
---

# Thank you

<small>Questions / metrics you’d like to see before rollout?</small>

<!-- End of deck -->
