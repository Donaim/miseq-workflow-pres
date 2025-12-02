---
title: MiSeq workflow
theme: seriph
background: https://raw.githubusercontent.com/Donaim/proviral-pres1/refs/heads/master/assets/forest.webp
info: |
  ## MiSeq Workflow
  Solutions and challenges in the modern pipeline
class: text-center
drawings:
  persist: true
mdc: true
zoom: 0.85
hideInToc: true
---
<br><br>
<h1 style='color:#20241f;opacity:85%;text-shadow:2px 2px 14px white'><b>MiSeq workflow</b></h1>
<h3 style='color:white;opacity:80%;text-shadow:1px 1px 0px black,-1px 1px 0px black,1px -1px 0px black,-1px -1px 0px black'>Solutions and challenges in the modern pipeline</h3>
<div class="absolute bottom-5 left-3" style='opacity:85%;text-shadow:1px 1px 0px black,-1px -1px 0px black'>
  <b>11 Dec 2025 · Lab Update</b>
</div>
<!-- Notes: Internal module; goal = increase usable-yield in denovo mode. -->

---

## Motivation

<DRAFT>
To provide understanding of how the pipeline works and why it works the way it does.
</DRAFT>

---

## The goal

<DRAFT>
To support these tasks:
- Transforming a V3 requisition into a resistance report.
- Transforming a research requisition into consensus sequence + intactness analysis.

In such a way that we have:
- Preservation of history in each. (keeping records and ability to recall important info)
- Continuous improvement for each. (things like feedback options and ease of software development)


Need a diagram here.
The diagram is a graph. Four nodes: two for requisitions, and two for the goals. Connected in the obvious way.
</DRAFT>

---

## Requisition

<DRAFT>
- How we get a requistion? Why we get it?
</DRAFT>

---

## Final reports

<DRAFT>
Show how they look like.
</DRAFT>

---

## The first split

<DRAFT>
Draw a diagram.
It's based on the previous diagram.
Instead of two parallel lines from requisitions to goals we have two joins and one vertical line.
The requisitions are joined by a node called "blood sample".
Then "blood sample" connects to "DNA sequence".
This "DNA sequence" is a node that similarly joins the two goals.

The interpretation is that in order to go from requisitions to goals we will get a hold of a sample and we will produce a DNA sequence.
</DRAFT>

---

## Subgoal 1: get blood

<DRAFT>
The obvious first state
</DRAFT>

<!-- - Omit interface descriptions: that's available in SOP's (TODO: provide SOP references/numbers). -->
<!-- - What are challenges here? -->
<!--   - Physical-informatics accounting coordination. -->
<!--   - Preservation and archival of traces (ORACLE DB). -->
<!--   - Mundane, I think? Mostly, I think, things like typos in the sample names. -->
<!--   - But QAI has come a long way to be in this position, I guess. -->

---

## Subgoal 2: produce consensus

<DRAFT>
Rationalize why an actual string of ACTG is so useful.

Note that going from sample to consensus is what we call "sequencing".
</DRAFT>

---

## Many (2) ways to do sequencing

<DRAFT>
Note that for sequencing there has been a lot of effort and standardization.

Explain two main choices that we have:
- Sanger sequencing (later consumed by ReCall).
- Next generation sequencing (later consumed by MiCall).

Explain tradeoffs and why for the goals we have, NGS is the better choice.
They include:
- need to sequence whole genomes, when Sanger can only give us up to ~700bp.
- high variability regions. especially indels make Sanger chromatographs interfere.
- NGS provides better quality scoring.
</DRAFT>

---

## NGS output

<NOTE>
The choice of NGS immediately gives us another intermediate state:
one when we get millions of short reads.

Draw a diagram here.
Should be a "zoomed in" picture: bounded by "sample -> DNA sequence" limit states.
Three nodes: "sample", "DNA sequence", and "millions of reads" inbetween them.
</NOTE>

---

## From sample to millions of reads

<NOTE>
I don't want to explain the challenges here because they are out of my competency as a programmer.

I still want to give appreciation to them.

My strategy is to list Nobel-level discoveries that were necessary to do this work:

- 
</NOTE>

---
layout: cover
background: https://cover.sli.dev
zoom: 2.0
hideInToc: true
---

# Thank you

<!-- End of deck -->
