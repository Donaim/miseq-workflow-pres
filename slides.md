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
<!-- End of deck -->

---
layout: cover
background: https://cover.sli.dev
zoom: 2.0
hideInToc: true
---

# Thank you

<!--

- That's all I have for today.

- This presentation will be available online.

- Thank you!

-->
