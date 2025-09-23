# Presentation Gap TODO

Goal: Augment `slides.md` for the Referenceless Contig Stitcher with deeper technical clarity while staying concise for internal approval.

## Must Add
1. Architecture Overview
   - Diagram / bullet of main data structures: Pool, ContigsPath, Overlap, Events.
   - Flow: Read contigs -> Seed paths -> Overlap eval -> Pool prune -> O2 greedy merge -> Output.
2. Constants & Threshold Rationale
   - MIN_MATCHES=99: chosen to exceed typical local alignment noise; ensures near-identity.
   - MAX_ALTERNATIVES=999 & intrapolate_number_of_alternatives formula (show n->cap curve conceptually).
   - SCORE_EPSILON sentinel role (coverage detection) & amplification (999, square) to avoid collision.
3. Path Pruning & Pool Mechanics
   - Pool bounded; raising minimum score as it fills.
   - Duplicate / dominated path avoidance.
   - Effect on runtime & safety (prevents combinatorial explosion).
4. Coverage & Edge Cases
   - Fully covered contig (score==SCORE_EPSILON) discard.
   - Slight insertion / micro-indel still merges if concordance remains above threshold.
   - Give-up condition: when top path is singleton and no strong overlaps remain.
5. Limitations & Risks
   - Ambiguity omission may under-merge complex repeats.
   - Fixed MIN_MATCHES may be suboptimal for very short contigs.
   - Potential to miss true joins when divergence > threshold.
   - Bias toward longer contigs (Length Prioritization) could overshadow rare variants.

## Should Add
6. Visual micro example (ASCII) of an overlap and concordance cut.
7. Highlight event taxonomy (Loaded, DeterminedOverlap, CombinedContigs, Covered, GiveUp) with purpose.
8. Metrics definition clarifications (acceptable bands for abstention rate, runtime overhead placeholder values X/Y once decided).

## Nice to Have
9. Performance note: typical runtime delta (placeholder until measured).
10. Future adaptive threshold sketch (dynamic MIN_MATCHES based on contig length / entropy).
11. JSON schema snippet for prospective structured event log.

## Slide Placement Plan
- Insert new core slides after current "How It Works (Condensed Rules)" and before "Outcomes".
  Order: Architecture → Constants & Thresholds → Path Pruning → Coverage & Edge Cases → Limitations.
- Append optional visuals either inside those slides (compact) or as mini-appendix if space tight.

## Acceptance Criteria
- All Must items appear as distinct, scannable slides/sections.
- No duplication with existing Appendices (E future enhancements kept lean; move overlapping items there if needed).
- Deck remains <= ~20 total slides including appendices.

## Open Questions (Fill During Edit)
- Final numeric targets (X, Y) once validation dataset stats available.
- Do we expose MIN_MATCHES as configurable? (Currently internal constant.)

---
Generated: 23 Sep 2025
