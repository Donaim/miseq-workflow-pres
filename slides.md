---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
background: https://raw.githubusercontent.com/Donaim/proviral-pres1/refs/heads/master/assets/forest.webp
title: Proviral pipeline
# some information about your slides (markdown enabled)
info: |
  ## Proviral pipeline
  Presentation for BC CfE
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: true
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
# transition: fade
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
zoom: 0.8
hideInToc: true
---

<br>
<br>

<h1 style='color: #191a12; opacity: 80%; text-shadow: 2px 2px 19px white;' > <b> Near-full-length HIV proviral amplification and sequencing pipeline </b> </h1>


<h2 style='color: white; opacity: 70%;
    text-shadow: 
    1px 1px 0px black,  /* Bottom-right shadow */
   -1px 1px 0px black,  /* Bottom-left shadow */
    1px -1px 0px black, /* Top-right shadow */
   -1px -1px 0px black, /* Top-left shadow */
    1px 0px 0px black,  /* Right shadow */
   -1px 0px 0px black,  /* Left shadow */
    0px 1px 0px black,  /* Bottom shadow */
    0px -1px 0px black; /* Top shadow */ ;
'> <b> by Aniqa Shahid and Vitalik Mysak </b> </h2>


<div class="absolute bottom-5 left-3 right-270" style='opacity: 80%;'>
   <b style='text-shadow: 
    1px 1px 0px black,  /* Bottom-right shadow */
   -1px 1px 0px black,  /* Bottom-left shadow */
    1px -1px 0px black, /* Top-right shadow */
   -1px -1px 0px black, /* Top-left shadow */
    1px 0px 0px black,  /* Right shadow */
   -1px 0px 0px black,  /* Left shadow */
    0px 1px 0px black,  /* Bottom shadow */
    0px -1px 0px black; /* Top shadow */ ;
'> 10 Oct 24 </b>
</div>

<!--

I am going to talk about the software part of the presentation today.
It will be quite technical, there is even going to be some code on the slides.

-->

---
hideInToc: true
---

# Second lab presentor

Vitalik Mysak

- Programmer, focusing on <br> MiCall and Proviral pipeline
- Contributor to free software projects
- Blogger at [https://vau.place](https://vau.place)
- Specialize in programming languages

<img v-drag="[464,18,443,443]" src="./assets/dog.jpg" />

<!--

First, I would like to introduce myself.

- My name is Vitalik, I am a software developer here. I am mostly working on MiCall and Proviral pipeline right now.

- I have a passion for programming, lots of hobby projects, and a programming blog.

- As you all know, the Russia-Ukraine war is still ongoing.

- As a Ukrainian, I am grateful for the support that many of you show for my country.

-->

---

# What is Proviral Pipeline?

- It is a collection of programs that output images and CSV files
- It is a software pipeline <!-- What this means is that it's a series of steps basically. -->
- It is, in large part, [HIVSeqInR](https://github.com/guineverelee/HIVSeqinR)/[HIVIntact](https://github.com/ramics/HIVIntact),
  <br> which is a program made by
  <br> [Guin Lee et al.](https://www.nature.com/articles/s41467-019-10659-2)/[Imogen Wright et al](https://retrovirology.biomedcentral.com/articles/10.1186/s12977-021-00561-5).
- It is a writing of
  <br> Daniel MacMillan, Don, Charlotte, and me <!-- I want to highlight the programmers' efforts. -->

<img v-drag="[450,169,363,363]" src="./assets/whatispipeline.webp" />

<SlideProgress />

<!--

- The Proviral Pipeline is essentially a suite of applications that transforms sequence data into analytical outputs, such as images and CSV files.
  <br> One of those outputs is the landscape plots that Aniqa showed previously.

- This suite is structured as a pipeline, which means that it processes data through in a sequence of steps.

- Big part of Proviral Pipeline is its defects detector. And, depending on the version, is either HIVSeqInR or HIVIntact.

- The development of the pipeline itself, is a collaborative effort -- involving Dan, Don, Charlotte, and myself.

-->

---
dragPos:
  catstructure: 447,98,431,431
---

<SlideProgress />

# Proviral sample workflow

- Data collection & MiCall processing
- Proviral pipeline startup
- HIVIntact processing
- Proviral pipeline outputs
  <br><sup>(such as regions clipping and landscapes)</sup>
- BBLabTools output

<img v-drag="'catstructure'" src="./assets/progress.webp" />

<!-- 

- Let's walk through the Proviral Sample Workflow. This is, basically, a standard progression for every proviral sample.
- We start with **data gathering and micall processing**. This is where biological, physical samples are turned into A-C-T-G strings, and then into more structured datasets.
- Next is **pipeline activation**, where the Proviral Pipeline comes alive. 
  <br> It loads the sample data, and also runs some **initial** checks and prepartions.
- After that HIVIntact takes stage. Its job is to check for any genetic anomalies present in the samples.
- Then we are back in the pipeline, as it gets the results from HIVIntact. It processes those results to provide a nicer summary.
- Finally, we use BBLabTools to make landscape plots. This step is actually manual, and optional, unlike all the rest.

-->

---
dragPos:
  miseq: 409,44,487,487
---

<SlideProgress />

# Initiation

<!-- TODO: better presentation form. More detail. -->

- Sample collection & loading

- Pickup by MiCall

- MiCall initial processing

Here is produces inputs to Proviral Pipeline

- MiCall starts proviral pipeline

<img v-drag="'miseq'" src="./assets/miseq.webp">

<!-- 

- This slide is also an overview, on what happens before

- As mentioned, we start with **Sample Collection**. This step was described in more detail by Aniqa. For us, it is important that it **can** fail. For example, it can result in sequences that look nothing like HIV.

- In any case, we progress to **MiSeq Loading**. Here samples are sequenced, which amounts to converting raw biological samples into digital data.

- Following this, **MiCall** takes over. The main task of MiCall it to assemble a long consensus seqeuence from short bits that come out of MiSeq.

While the HIV full length sequence is 9000 base pairs, it is assembled from pieces that are at most 300 base pairs long.

- Once that's done, we get to the **Output Generation** phase.

These outputs contain the raw, full length sequences, as well as various statistics about them.

- Finally, the proviral pipeline is started by MiCall.

-->

---
dragPos:
  pipeline: 500,49,450,450
---

<SlideProgress />

# Pipeline structure

- Proviral pipeline loads data

From files generated by MiCall <br>
<small>(specifically `conseq.csv`, `contigs.csv`, and `cascade.csv`)</small>

- Proviral pipeline runs QC (quality control)

- Proviral pipeline trims primers off

- Samples that pass all checks proceed to HIVIntact
<br><sup>HIVSeqInR is also still supported</sup>

- Landscape data is generated

- Final report is generated

<img v-drag="'pipeline'" src="./assets/pipelines.webp">

<!--

- When proviral pipeline is started, it first loads the data it needs, from files produced by MiCall.

- It then runs a series of quality checks on the samples that it's just loaded.

- It also then trims primers. I will talk about this step in detail on a following slide.

- All samples that pass the initial checks, are then analyzed for defects by HIVIntact.

- Once that's done, the proviral pipeline summarizes the outputs of HIVIntact. This is, in particular, for the generation of landscape plots.

-->

---
hideInToc: true
dragPos:
  checkqc: 477,36,490,490
---

<SlideProgress />

### Proviral QC is a series of steps:

<br>
<br>

<AccordionQC />

<img v-drag="'checkqc'" src="./assets/qc1.webp" >

<!-- 

- There are multiple quality checks that the proviral pipeline runs on every sample.

- Generally, the idea is to discard any sequences that have evidence of not correctly representing the actual virus population.
  
  That could happen due to sequencing errors, which can often be detected by these checks.
  
- So, for instance, let's take a look at the low coverage check from this slide.

`*click on Low Internal Coverage*`

- This check is about regions that have low coverage. Such regions are marked with character X in the input sequence.
  They fail QC, and are skipped, because for these regions we don't really know what is the true DNA sequence there.

-->

---
dragPos:
  primers: 15,125,387,258
  bottom: 39,374,461,97
  left: 415,25,552,537
---

<SlideProgress />

# Primer trimming

<v-drag pos="left">

The pipeline proceeds to find primers in the remaining sequences.
There are two primers: one at 5’ (forward) and one at 3’ (reverse) end.

<AccordionPrimers />

</v-drag>
<v-drag pos="bottom">

Once primers are found, they are cut off.

</v-drag>

<img v-drag="'primers'" src="./assets/cutoff.png" >

<!--

- Unlike the low coverage X's, primers are something that is present in the actual, physical input sequence.

- They are short strings of nucleotides that are purposefully added to the both ends of the amplified sequences.

- This serves two purposes: it allows the rybosome to do the amplification, and they are evidence of successfull assembly.

- Recall that during assembly, which happens in MiCall, we put short sequences of DNA together, to recover the full length consensus.

If it happens that the primers do not end up in correct places, meaning at the both ends, then something in that process went wrong.

- Now, confirming the presence of primers is a little bit more tricky than it seems. That's why this slide has 4 points on the right.

`*click on the Reverse Complementation*`

- For example, the pipeline tries to handle a common case where the "reverse complement" of input sample was sequenced.

This point describes that.

- Finally, when primers are found, they are removed, because they are something that we added synthetically.

-->

---
dragPos:
  cfeintact: 527,14,437,599
  cfeintactwebsite: 469,-26,567,646
---

<SlideProgress />

# HIVIntact

- Performs automated checks for HIV-1 genome integrity
- Evolved into CFEIntact
- Identifies various genetic anomalies:
  - Large deletions
  - Hypermutations
  - Scrambles
  - ...
- Generates reports for regions, defects, <br> and various holistic properties
- Details at https://cfe-lab.github.io/CFEIntact

<v-drag pos="cfeintactwebsite">
<iframe src="https://cfe-lab.github.io/CFEIntact/" style="transform: scale(0.8); margins: 1px; width: 100%; height: 100%; border: 2px solid green; overflow: auto;"></iframe>
</v-drag>

<!-- <img v-drag="'cfeintact'" style='border: 2px solid green;' src="./assets/cfeintactdocs.png" > -->

<!--

2. **Key Functionality**:

HIVIntact provides an automated discovery of genetic defects from a DNA sequence.

2. **Introduction**:

In the lab, we use a program called CFEIntact.

That program is our modified version of HIVIntact.

We modified it to the point where the original publication was no longer correct for our version of HIVIntact.

So changed the name, as is done in such cases.

3. **Main Anomalies Detected**:

CFEIntact looks for defects that can severely impact genome functionality.

Hypermutations, are one example, but there are many possible kinds of them, and each defect type requires a custom algorithm to detect.

Because of this, CFEIntact can easily take up another presentation, so it's best to just check out the documentation website.

In particular, I imagine most people would be interested in the workflow page of the docs.

It describes the individual algorithms and defect code they produce.

4. **Output Files**:

For every sample, CFEIntact generates detailed reports in CSV format. They capture defects, regions, and other relevant data.

-->

---

<img src="./assets/defects.png" />

<!--

- On this slide, I have examples of common defects.

- And premature stop codons, illustrated here, are particularly common.

- They are defects caused by mutation in HIV that introduces a stop codon, such as T-A-G, in the middle of a gene.

- Because of this mutation, the rybosome will only transcribe some part of the gene. So, it practically kills it.

-->

---

<SlideProgress />

# Back to Proviral Pipeline

<!-- TODO: better presentation form. More detail. -->

- Analyzes CFEIntact's outputs.
- Decides what single error to report.
- Slices the input sequence into its parts.
- Generates `landscapes.csv`

<img v-drag="[247,256,459,262]" src="./assets/windowsxp.webp" />

<!--
When CFEIntact finishes, it's outputs go back into the proviral pipeline.

The pipeline summarizes them, and it also performs some additional processing,
 in particular it extracts some regions of interest from the input sequence,
 and it generates data for landscape plots, the csv file.
-->

---

<SlideProgress />

# Region extraction in Proviral Pipeline

- Aligns sequences to the **HXB2 reference genome**.
- Extracts key **HIV regions**: *gag*, *pol*, *env*, *psi_sl1*, *gp120*, *ltr3*, etc.
- Handles **inversions** and **defective regions**.
- Outputs raw sequences to `table_precursor.csv`.

<img v-drag="[613,184,366,366]" src="./assets/regions.webp" />

<!--

Now few words on how the pipeline extracts genomic regions from the sequences.

- The first step is **aligning the input sequences** to the well-known **HXB2 reference genome**. For this, the pipeline uses blast and minimap2 programs.
 
- Once aligned, the pipeline extracts key **HIV regions** like *gag*, *pol*, and *env*. But also some smaller, non-coding regions such as *psi_sl1*, *2*, and *3* and *ltr3* and *ltr5*.
 
- During this process, the pipeline is also looking for **inversions** and **defective regions**. If any region is inverted or missing key parts, it gets logged to standard error.
 
- All this information is then included into the final output file called `table_precursor.csv`.

-->

---
dragPos:
  landscapes: 19,-503,853,924
---

<SlideProgress />

# Pipeline's final verdict

<div></div>

- Proviral pipeline receives a _list of defects_ from **CFEIntact**.

- It tries to determine the most severe defect, <br>which becomes the pipeline's verdict.

- This procedure is described by the decision tree 👉

<v-drag pos="landscapes">

```mermaid
flowchart TD
    Q{ } --> W[NonHIV]
    Q{ } --> E{ }
    E --> R[LongDeletion]
    E --> T{ }
    T --> Y[InternalInversion]
    T --> U{ }
    U --> I[Scramble]
    U --> O{ }
    O --> P[APOBECHypermutationDetected]
    O --> A{ }
    A --> S[MajorSpliceDonorSiteMutated]
    A --> D{ }
    D --> F[PackagingSignalDeletion]
    D --> G{ }
    G --> J[PackagingSignalNotComplete]
    G --> K{ }
    K --> L[RevResponseElementDeletion]
    K --> Z{ }
    Z --> X[WrongORFNumber]
    Z --> C{ }
    C --> V[DeletionInOrf]
    C --> B{ }
    B --> N[InsertionInOrf]
    B --> M{ }
    M --> A1[InternalStopInOrf]
    M --> A2{ }
    A2 --> A3[FrameshiftInOrf]
    A2 --> A4[Intact]
```

</v-drag>

<!--
CFEIntact reports every defect it can find.

But it is often sufficient, and easier to look at, the most severe one.

The pipeline determines that based on its builtin decision tree.

It is displayed here on the slide.

`*drag the tree*`

For example, it first checks whether there is a defect called *NonHIV*.

Then if there is a long deletion.

And so on.

It outputs the first error found down this chain.

So this is more of a list than a tree.
-->

---
dragPos:
  bblabs: 160,151,675,399
---

<SlideProgress />

# BBLabTools

- Useful tools for HIV research from the Brockman and Brumme Lab at SFU.

<img v-drag="'bblabs'" src="./assets/bblabs1.png">

<!--
- BBLabTools is a website that hosts various research tools.

- It was written by SFU and CFE staff.

- The website is hosted on our internal network, the link is here on the slide.
-->

---
hideInToc: true
dragPos:
  landscapes: 566,6,349,548
  defecttype: 94,180,470,365
---

<SlideProgress />

# BBLabTools

- Draws landscape plots (dev version).

- There is a translation step involved:

<v-drag pos="defecttype">
```python {*}{maxHeight: '100%'}
DEFECT_TYPE = {'LargeDeletion': 'Large Deletion',
               'LongDeletion': 'Large Deletion',
               'InternalInversion': 'Inversion',
               'ScramblePlus': 'Scrambled',
               'ScrambleMinus': 'Scrambled',
               'ScrambleCheck': 'Scrambled',
               'Scramble': 'Scrambled',
               'Hypermut': 'Hypermutated',
               'APOBECHypermutation': 'Hypermutated',
               'Intact': 'Intact',
               'Inferred_Intact': 'Intact',
               'PrematureStop_OR_AAtooLong_OR_AAtooShort': 'Premature Stop',
               'PrematureStop_OR_AAtooLong_OR_AAtooShort_GagNoATG': 'Premature Stop',
               'Inferred_PrematureStopORInframeDEL': 'Premature Stop',
               'Inferred_PrematureStopORInframeDEL_GagNoATGandFailed': 'Premature Stop',
               'Inferred_PrematureStopORInframeDEL_GagNoATG': 'Premature Stop',
               'InternalStop': "Premature Stop",
               'MutatedStopCodon': "Premature Stop",
               'MutatedStartCodon': "Premature Stop",
               'SequenceDivergence': "Divergence",
               'Deletion': "Premature Stop",
               'Insertion': "Large Insertion",
               'Frameshift': "Frameshift",
               '5DEFECT': "5' Defect",
               '5DFECT_IntoGag': "5' Defect",  # this is a typo in HIVSeqinR
               '5DEFECT_GagNoATGGagPassed': "5' Defect",
               '5DEFECT_GagNoATGGagFailed': "5' Defect",
               'Inferred_Intact_GagNoATG': "5' Defect",
               'Inferred_Intact_NoGag': "5' Defect",
               'Intact_GagNoATG': "5' Defect",
               'MajorSpliceDonorSiteMutated': "5' Defect",
               'PackagingSignalDeletion': "5' Defect",
               'PackagingSignalNotComplete': "5' Defect",
               'RevResponseElementDeletion': "5' Defect",
               'NonHIV': 'Chimera',
               'AlignmentFailed': 'Chimera',
               'UnknownNucleotide': 'Chimera',
               }
```
</v-drag>

<img v-drag="'landscapes'" src="./assets/landscapes1.svg">

<!--
We use development version of BBLabs to draw landscape plots.
Eventually this version will be released, and everybody will be able to make these drawings themselves.

Before the plots are generated, a translation step converts the defect types into categories that BBLabTools can represent graphically.

This plotting tool supports both CFEIntact and HIVSeqInR.
-->

---

<SlideProgress />

# Summary of output files

<br>

<AccordionOutputsSummary />

<!-- 

On this slide I have prepared a description of individual files that
you as users of the pipeline are interacting with.

The list is in the order of importance.

Let's take `defects.csv` for example `*click on defects.csv*`.

This file describes individual defects detected by CFEIntact.
Here is where to find it `*point at the path*`.

And here I have example defects `*open example*`.

The structure of it is straightforward, but there is usually a lot of data here.

-->

---

<SlideProgress />

# Future plans

<!-- TODO: better presentation form. More detail. -->

- BBLabTools release.
- Switch to CFEIntact.
- Better support for subtypes in CFEIntact.
- More regions recognition in CFEIntact.
- Better output description in CFEIntact.

<img v-drag="[452,85,425,425]" src="./assets/future.webp">

<!--

We have some plans for the future of the pipeline.

- As I mentioned, BBLabTools will be released soon, with the landscape plots support.

- Today I presented the pipeline as if it was already using HIVIntact. But that is only coming with the next release of the pipeline. The current production version of the pipeline uses HIVSeqInR.

- CFEIntact is currently not doing well with subtypes other than B, and we want to fix that.

- It would be nice to output more regions information in CFEIntact, such as the various PSI locations. Basically to completely overtake that function from the proviral pipeline.

- We want to give CFEIntact a notion of "confidence" and "severity", with which every defect can be labelled. 
  This can help researchers decide if they need to take a closer look.
  A natural example is the hypermutation check, which outputs probability that a given sequence was hypermutated.
  That could be our confidence value.
-->

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
