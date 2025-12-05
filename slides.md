---
title: MiSeq Workflow
theme: seriph
background: https://raw.githubusercontent.com/Donaim/proviral-pres1/refs/heads/master/assets/forest.webp
info: |
  ## MiSeq Workflow
  Solutions and challenges in the modern pipeline
class: text-center
drawings:
  persist: true
mdc: true
zoom: 1.2
hideInToc: true
---

<br>
<h1 style='color:white;opacity:90%;text-shadow:0px 0px 24px white'><b>MiSeq Workflow</b></h1>
<h3 style='color:white;opacity:80%;text-shadow:1px 1px 0px black,-1px 1px 0px black,1px -1px 0px black,-1px -1px 0px black'>Solutions and challenges in the modern pipeline</h3>
<div class="absolute bottom-5 left-3" style='opacity:85%;text-shadow:1px 1px 0px black,-1px -1px 0px black'>
  <b>11 Dec 2025 · Lab Update</b>
</div>

<!--
Hello everyone.

In this talk I want to walk through the MiSeq-based workflow that takes us from a requisition to a resistance report and an intactness analysis.

I am not going to give a step-by-step SOP or a MiSeq troubleshooting guide. Instead, I want to focus on why the pipeline is shaped the way it is, and why we chose the particular intermediate states that we did.

Think of this as a map of the territory. After this talk, my goal is that when you hear “MiSeq run” or “consensus” or “intactness”, you have a clear sense of where that sits in the bigger picture and why we bother doing it in the first place.
-->

---

## Motivation

Provide understanding of **how** the MiSeq pipeline works and **why** it is built the way it is.

- Every report we sign off on passes through many invisible steps.
- Each step encodes assumptions and constraints that affect what the result means.
- Understanding the key “states” of the pipeline helps us trust the output, debug problems, and argue for or against changes.

<!--
The starting question for this talk is very simple: why think about the MiSeq workflow at all?

Every resistance report or intactness analysis that we produce has travelled through a long chain of invisible steps. If we only ever see one piece of that chain, it is hard to judge how robust the final answer really is.

Each stage of the pipeline encodes assumptions and constraints. For example, what kind of sample we accept, what region of the genome we target, what we do when coverage is low, all of that quietly shapes the interpretation.

By focusing on the key intermediate “states” of the pipeline, rather than every technical detail, we can reason about the behaviour of the whole system. That helps us trust the output when everything works, spot when something feels off, and have more grounded discussions about changing or extending the workflow.
-->

---

## The goal

<!--
TODO:
- add a diagram. the diagram should have four nodes and two edges:
  - V3 requisition -> resistance report
  - research requisition -> intactness results
-->

Two main transformations:

- Turn a **V3 requisition** into a **resistance report**.
- Turn a **research requisition** into a **consensus sequence plus intactness analysis**.

And do this in a way that:

- Keeps the **history** of each sample and run reconstructable.
- Makes it easy of use and maintain.

<!--
If we compress all the complexity down to just a couple of sentences, the pipeline is trying to do two main things.

First, for clinical V3 testing, we start from a V3 requisition and we owe the clinician a resistance report that they can use.

Second, for research work, we start from a research requisition and we owe our collaborators intactness analyses that they can trust in their projects.

Around those two transformations we have two design principles. The first is history: we want to be able to reconstruct what happened to a sample or a run months or years later. That means we care about preserving inputs, intermediate artefacts, and outputs, rather than just the final PDF.

The second is continuous improvement: we know methods, software, and standards change. We do not want a pipeline that collapses every time we touch it. So we aim for a structure where we can add QC, replace tools, or change thresholds, without losing track of what we did before.
-->

---

## Requisition

### Why start with a requisition?

- It expresses **why** we are doing any work at all for this sample.
- It ties a **person or project** to a **specific promise**: resistance, consensus, intactness, or some combination.
- It defines the **obligations** downstream: how carefully we must track, interpret, and report.

### What it carries conceptually

- The clinical or research question we are answering.
- The type of sample that will be sent.
- The type of pipeline and report that the MiSeq workflow should produce.

<!--
The first state in the workflow is the requisition.

We start here because the requisition is where the “why” of the whole process is written down. Someone is asking us to do something specific for a person or for a project, and the requisition is the formal record of that request.

Conceptually, the requisition ties three things together. It ties a person or a coded participant ID, it ties the kind of sample that will arrive in the lab, and it ties the type of answer we promise to return, like a resistance interpretation or an intactness analysis.

The way we handle the sample downstream depends on what is written here. A clinical V3 requisition implies different obligations, timelines, and reporting expectations than a purely research requisition, even if the MiSeq run in the middle looks similar.

So in the state diagram that we will use later, the requisition is the starting node. From that starting point, the very first subgoal is to turn this abstract request into an actual sample tube we can work with, and eventually into a DNA sequence and a report that honours the original question.
-->

---

## Final reports

<!--
TODO: Say something general about these reports. Ie what they contain, why they are useful.
-->

TODO: Show how they look like.

---

## The first split

<!--
TODO:
Draw a diagram.
It's based on the previous diagram.
Instead of two parallel lines from requisitions to goals we have two joins and one vertical line.
The requisitions are joined by a node called "physical sample".
Then "physical sample" connects to "DNA sequence".
This "DNA sequence" is a node that similarly joins the two goals.
-->

We can describe the whole workflow as passing through two shared internal states:

- A **physical sample** that has actually arrived in our lab.
- A **consensus DNA sequence** that represents the virus in that sample.

Both clinical and research work follow the same backbone:

> V3 / research requisition → **physical sample** → **DNA sequence** → resistance / intactness report

- The details differ at the edges, but almost everything we care about happens around these two states.
- So the rest of the talk will treat “sample” and “consensus” as the main anchors of the MiSeq pipeline.

<!--
Up to now I’ve shown you the overall goals: starting from a requisition and ending with either a resistance report or an intactness analysis.

In between those endpoints there are a lot of individual steps: lab work, sequencing chemistry, file transfers, bioinformatics, quality control, data upload.

The first ovious subgoal is to get a physical sample into our lab. The second subgoal is to turn that sample into a consensus DNA sequence.

Once you accept those two states as the backbone, both the V3 clinical pipeline and the research pipeline look like variations on the same theme. Different requisitions, different reports, but the same two internal states in the middle.
-->

<div></div>

---

## Subgoal 1: get a physical sample

What this state means

- An actual **tube in our lab**, labelled and received under a known requisition.
- Enough **material of the right type** (plasma, whole blood, PBMCs, etc.) to run the planned assays.
- A sample whose **identity and timing** we trust.

It is its own goal

- Without a sample, the rest of the pipeline is purely hypothetical.
- This is where the real world and our digital systems first have to agree on **who**, **what**, and **when**.
- Mistakes here — wrong person, wrong tube, wrong date — are worse than having no sample at all.

<!--
The first subgoal sounds almost too obvious: we need to get a physical sample.

But conceptually this is a big moment in the workflow. Up until now we only had a requisition, which is essentially a promise on paper: someone is asking us to do something for a particular person or project.

This state carries hidden assumptions. The type of sample determines what biology we can and cannot see. Plasma, whole blood, or PBMCs will all give us different windows into the virus and the host.

And identity is crucial. A beautifully sequenced sample from the wrong person is worse than no sequence at all. So at this point in the workflow we care a lot about matching the tube in the rack to the digital record in our systems.
-->

---

## Subgoal 2: produce consensus

What this state means

- We have an complete, digital representation of the virus from that sample.
- An actual string of ACTG letters.

Why a consensus sequence is such a useful state

- A representation that is **easy to store, compare, and interpret**.
- It is the **standard interface** to everything downstream: HIVdb, intactness tools, alignments, and reporting.
- It collapses a very complicated experiment into a simple string of A, C, T, and G that we can reason about.
- It lets us **re-interpret** the same data later as rules, tools, and guidelines change.

<!--
The second subgoal is to produce a consensus DNA sequence.

This consensus becomes the standard interface to the rest of the pipeline. It is what we align to reference, what we feed into HIVdb for resistance interpretation, and what we send into the intactness pipeline for defect classification.

The reason we treat "consensus sequence" as its own goal is that it separates two worlds. Upstream we worry about chemistry, instruments, and file transfers. Downstream we worry about biological interpretation. The consensus sequence is the bridge between the two, and because it is such a powerful bridge we invest a lot of effort in getting it right and in keeping the raw data available behind it.
-->

---

## Counterfactual pondering

<!--
More on this: https://chatgpt.com/share/692f7e0d-e8d8-800e-80bc-30cc076cbec6
-->

What partial goals we could still achive without Subgoal 1 and Subgoal 2?

<!--
What partial goals we could still achive without Subgoal 1 and Subgoal 2?

There are ways to quantify intactness without ever seeing a DNA sequence. Assays like IPDA use droplet digital PCR with a couple of probes to count how many proviruses look “intact enough” at those sites, and they never produce an ACTG string.

More of a stretch, but in principle we can even learn something relevant about the host’s DNA without ever receiving a sample from that person. You know how blood type can sometimes be deduced from the parents’ blood types? The same logic applies to the HLA allele that carries abacavir risk: if neither parent has that problematic allele, then their children can’t inherit it either.
-->

---

## From requisition to physical sample

A requisition arrives → lab receives a sample → information must flow to bioinformatics pipeline

**Key challenges:**

- **Real-world ↔ Digital coordination**: Physical tubes, in various steps along the pipeline, must match digital records in the system
- **Historical traceability**: Every sample, every run, every decision must be reconstructable months or years later
- **Data integrity**: Typos in sample IDs, missing metadata, or mismatched records can derail the entire pipeline

**Our solution: QAI**

Our in-house Laboratory Information Management System that bridges the physical and digital worlds.

<!--
This is the first critical handoff in our workflow - moving from an abstract request to a concrete physical sample that both the lab and our automated systems can track.

The challenge here is coordination. Someone in the lab is handling a physical tube with a label. Meanwhile, our bioinformatics pipeline needs to know exactly which sample that tube represents, what project it belongs to, and what kind of analysis we promised to deliver.

QAI is our custom-built LIMS - Laboratory Information Management System. Here, it solves three core problems:

First, it provides the coordination layer between the physical lab and our automated informatics systems.

Second, it handles historical preservation. QAI stores everything in an Oracle database. This means years later, if someone asks "what happened with this sample?", we can reconstruct many details.

Third, it provides input validation. Before a sample ever reaches the sequencer, QAI checks that sample IDs are formatted correctly, that the requisition exists, that the sample type matches what was requested. This catches problems early, before they waste expensive sequencing runs.

The concrete artifact that QAI produces is the sample sheet - a CSV file that defines which samples are in a MiSeq run and how they should be processed. Both MiSeq software and MiCall's automated watcher will later read those sample sheets.

The procedure details - how lab staff use QAI's interface, what fields they fill in - those are documented in our SOPs. What matters here is understanding QAI's architectural role: it's the authoritative source of truth that connects requisitions to samples to sequencing runs.
-->

---

## Many (2) ways to do sequencing

<NOTE>
Note that going from sample to consensus is what we call "sequencing".

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
one when we get millions of short reads, represented as a "FASTQ" file.

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
- <10Kbp in a sea of 4Gbp of DNA.
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

## From millions of reads to consensus

<NOTE>
This is the main part, basically.

Need like an introduction.
</NOTE>

---

## Transport

<NOTE>
Challenge: actually getting hold of the files with millions of reads.

Want this to be automatic (ie not a user clickling and moving files on a flash drive or something).
Need to be reliable beyond just automatization.

Solution:
Basically, it's done via MiSeq itself.
MiSeq has a computer that runs Windows.
This computer has a little daemon script, put there by Illumina people.
We configured the script to copy data from the machine to our network connected `RAW_DATA` drive.
</NOTE>

---

## Discovery

<NOTE>
Challenge: how do we know when data is ready?

MiSeq itself doesn't notify us in a scripted way (no any kind of API).

Solution:
We have an periodic task that runs in ScriptBunny (every 15mins).
It monitors `RAW_DATA` drive for new files and folders.
It looks if those files "look done copying", and then puts a "flag file" called `needsprocessing`.

Then we have a periodic tasks in MiCall watcher (every 10mins) and in miseqqc (every 10 mins).
They look for `needsprocessing` file to do anything.
</NOTE>

---

## Many (2) ways to build consensus

<DRAFT>
- Denovo assembly
- Remapping approach
</DRAFT>

<NOTE>
Show a graphic of millions of reads.
It should show depict that these reads are short and they come from random places in the virus' genome.

Why these two ways for consensus? What are other alternatives?
</NOTE>

---

## Remapping

<NOTE>
In this mode we take the millions of reads and map them to a reference (HXB2-like) genome.
We employ `bowtie2` for this.

Challenges:
- ???

Solutions:
- ???
</NOTE>

---

## Remapping [2]

<NOTE>
Show graphically the algorithm.
</NOTE>

---

## Denovo assembly

<NOTE>
Challenges:
- Overlap finding is very computationally hard. TODO: estimate run time of a naive algorithm.

Solutions:
- Hash-table (k-mer) based overlap counting.
- Stitching (both reference-free and reference-based stitching is performed in MiCall)
</NOTE>

---

## Denovo assembly [2]

<NOTE>
Show graphically the algorithms.
</NOTE>

---

## Pipeline orchestration

<NOTE>
Challenge:
- Want to preserve all inputs and outputs for historical and debugging reasons.

Solutions:
- Run all important steps through Kive.
- Poll Kive to see when results are available.
- Download results to `RAW_DATA`.
</NOTE>
---

## Quality control

<NOTE>
Explain how Phred scores are calculated in MiSeq.
Then explain how MiCall is filtering out bad reads.

Then explain generation of coverage and concordance plots.
These plots display how well reads mapped to the contigs.
</NOTE>

---

## MiseqQC processing

<NOTE>
Explain how `MiseqQCReport` perl script daemon goes through and uploads the Phred scores to Oracle database.

Explain how then `MiseqQC` produces a PDF report and uploads it to http://192.168.69.223/MiSeq_QC/01-Aug-2025.M01841/
</NOTE>

---

## QAI upload

<NOTE>
We upload quality control data to QAI for easy display.
Users can see the a summary of individual reads quality scores and coverage plots.
</NOTE>

---

## From consensus to reports

<NOTE>
Reminder, we produce two kinds of reports:
- V3 resistance interpretation
- intactness analysis
</NOTE>

---

## Proviral pipeline

<NOTE>
Challenge:
- given a consensus sequence, determine reproduction intactness of the virus and categorize various types of defects.

Solution:
- `CFEIntact` - defects detector.
- `proviral` - a pipeline with its own QC and prep steps. calls `CFEIntact` as part of its workflow.
- `BBLabs/alldata/bblab_site/tools/proviral_landscape_plot/` - a tool where users manually upload outputs of `proviral` to visualize defects distribution.
</NOTE>

---

## CFEIntact

<NOTE>
Challenge: find open reading frames.

Solution:
- Align consensus to reference genome.
- Look for start codon (ATG).
- Look for stop codons (TAA, TAG, TGA).
- Find the longest ORF between start and stop codons.

</NOTE>

---

## CFEIntact [2]

<NOTE>
Challenge: determine if an ORF is intact.

Solution:
- Look for premature stop codons.
- Look for frameshifts.
- Look for deletions beyond a threshold.
</NOTE>

---

## CFEIntact [3]

<NOTE>
Other challenges:
- Detect hypermutation.
- Determine if packaging signal is intact.
- Determine if major splice donor site is intact.
- Detect out-of-order genes.
- Detect large deletions.
</NOTE>

---

## Resistance interpretation

<NOTE>
Challenge:
- given a consensus sequence, determine if the original virus is resistant to given drugs

Solution:
- align consensus to reference genome
- then use HIVdb to score resistance
</NOTE>

---
layout: cover
background: https://cover.sli.dev
zoom: 2.0
hideInToc: true
---

# Thank you
