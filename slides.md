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
</DRAFT>

<NOTE>
Need a diagram here.
The diagram is a graph. Four nodes: two for requisitions, and two for the goals. Connected in the obvious way.
</NOTE>

---

## Requisition

<NOTE>
- How we get a requistion? Why we get it?
</NOTE>

---

## Final reports

<NOTE>
Show how they look like.
</NOTE>

---

## The first split

<NOTE>
Draw a diagram.
It's based on the previous diagram.
Instead of two parallel lines from requisitions to goals we have two joins and one vertical line.
The requisitions are joined by a node called "blood sample".
Then "blood sample" connects to "DNA sequence".
This "DNA sequence" is a node that similarly joins the two goals.

The interpretation is that in order to go from requisitions to goals we will get a hold of a sample and we will produce a DNA sequence.
</NOTE>

---

## Subgoal 1: get blood

<NOTE>
The obvious first state
</NOTE>

<!-- - Omit interface descriptions: that's available in SOP's (TODO: provide SOP references/numbers). -->
<!-- - What are challenges here? -->
<!--   - Physical-informatics accounting coordination. -->
<!--   - Preservation and archival of traces (ORACLE DB). -->
<!--   - Mundane, I think? Mostly, I think, things like typos in the sample names. -->
<!--   - But QAI has come a long way to be in this position, I guess. -->

---

## Subgoal 2: produce consensus

<NOTE>
Rationalize why an actual string of ACTG is so useful.

Note that going from sample to consensus is what we call "sequencing".
</NOTE>

---

## Counterfactual pondering

<DRAFT>
What partial goals we could still achive without Subgoal 1 and Subgoal 2?

There are ways to quantify intactness without ever seeing a DNA sequence. Assays like IPDA use droplet digital PCR with a couple of probes to count how many proviruses look “intact enough” at those sites, and they never produce an ACTG string.

More of a stretch, but in principle we can even learn something relevant about the host’s DNA without ever receiving a sample from that person. You know how, in simple Mendelian genetics, blood type can sometimes be deduced from the parents’ blood types? The same logic applies to the HLA allele that carries abacavir risk: if neither parent has that problematic allele, then their children can’t inherit it either.

</DRAFT>

<NOTE>

(More on this: https://chatgpt.com/share/692f7e0d-e8d8-800e-80bc-30cc076cbec6 )
</NOTE>

---

## Many (2) ways to do sequencing

<NOTE>
Note that for sequencing there has been a lot of effort and standardization.

Explain two main choices that we have:
- Sanger sequencing (later consumed by ReCall).
- Next generation sequencing (later consumed by MiCall).

Explain tradeoffs and why for the goals we have, NGS is the better choice.
They include:
- need to sequence whole genomes, when Sanger can only give us up to ~700bp.
- high variability regions. especially indels make Sanger chromatographs interfere.
- NGS provides better quality scoring.
</NOTE>

---

## Subgoal 3: NGS output

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
</NOTE>

---

## Needle in a haystack

<NOTE>
I want to highlight the basic challange that is understandible to me as programmer.
Namely the "needle in a haystack" kind of problem with DNA.
We try to find something really tiny.

Give a perspective of sizes.
Ie in 1ml of blood sample we have:
- 5-6×10⁹ number of cell-like particles.
- 7.5×10⁶ of them are white-blood cells.
- ~100 of them are infected cells.
- ~1 copy of HIV RNA.

(This is in a ART-supressed person)
(More on this: https://chatgpt.com/share/692f7528-ba64-800e-9340-113d71ef2534 )
</NOTE>

---

## Sequencing solutions

<NOTE>
My strategy here is to list Nobel-level discoveries that were necessary to do this work.

First nobels provide the absolute basic: "what and where to look for".

- 1975 Physiology or Medicine – Dulbecco, Temin, Baltimore. Awarded for discoveries on how tumour viruses interact with cellular genetic material, including the discovery of reverse transcriptase and the provirus concept.
- 2008 Physiology or Medicine – Barré-Sinoussi & Montagnier (plus zur Hausen for HPV) “for their discovery of human immunodeficiency virus”  That’s the step where “some immune-destroying thing” becomes this specific retrovirus with this specific genome that you are now sequencing on MiSeq.
- 1993 Chemistry – Mullis & Smith. One half to Kary Mullis “for his invention of the polymerase chain reaction (PCR) method”; the other to Michael Smith for oligo-based site-directed mutagenesis.
- 1980 Chemistry – Berg, Gilbert, Sanger. Half to Walter Gilbert and Frederick Sanger “for their contributions concerning the determination of base sequences in nucleic acids.” MiSeq is based on Sanger.

(More on this: https://chatgpt.com/share/692f7564-8780-800e-88e8-f7d33366fc3c )
</NOTE>

<DRAFT>
"What’s inside this MiSeq run?"

- Enzymes first isolated in work that won the 1959 Nobel
- A reverse-transcription concept that won the 1975 Nobel
- A sequencing concept that won the 1980 Nobel
- An amplification concept that won the 1993 Nobel
- A virus discovered in work that won the 2008 Nobel
- Plus Illumina’s own chemistry, optics, microfluidics and control software to make all of the above behave like a push-button black box.
</DRAFT>

---
layout: cover
background: https://cover.sli.dev
zoom: 2.0
hideInToc: true
---

# Thank you

<!-- End of deck -->
