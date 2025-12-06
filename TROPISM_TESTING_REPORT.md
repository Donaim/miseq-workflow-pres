# HIV Tropism Testing via MiCall

## Executive Summary

**What is tropism testing?** It determines which coreceptor HIV uses to enter cells - CCR5 (R5-tropic) or CXCR4 (X4-tropic). This is clinically important for prescribing CCR5 antagonist drugs like maraviroc.

**MiCall's role:** The lab uses MiCall for clinical V3 loop tropism testing. This is the ONLY clinical application of MiCall (besides research). ReCall handles general resistance testing.

---

## Theoretical Background

### What is HIV Tropism?

HIV enters human cells by binding to CD4 and then to a **coreceptor**. There are two main coreceptors:

- **CCR5** - chemokine receptor normally involved in immune response
- **CXCR4** - another chemokine receptor

HIV variants can be classified by which coreceptor they use:

- **R5-tropic** (CCR5-using) - most common, especially early in infection
- **X4-tropic** (CXCR4-using) - can emerge during disease progression
- **Dual-tropic** - can use either coreceptor

### Why Does This Matter Clinically?

**Maraviroc** is a CCR5 antagonist drug. It blocks the CCR5 coreceptor, preventing R5-tropic HIV from entering cells.

**Critical point:** Maraviroc ONLY works if the patient's virus is R5-tropic. If they have X4-tropic virus, maraviroc is useless and could lead to treatment failure.

**Before prescribing maraviroc**, clinicians MUST test tropism to ensure the virus uses CCR5.

### The V3 Loop Connection

The HIV envelope protein has a region called the **V3 loop** (Variable region 3). This loop is the primary determinant of coreceptor usage.

**Key insight:** Specific amino acid sequences in the V3 loop predict which coreceptor the virus will use.

- Certain amino acids at specific positions favor CCR5 binding → R5-tropic
- Other patterns favor CXCR4 binding → X4-tropic

This means we can predict tropism by sequencing the V3 loop and analyzing the amino acid sequence.

---

## The Geno2Pheno Algorithm

### What is Geno2Pheno (G2P)?

**Geno2pheno** is an algorithm that predicts phenotypic tropism (R5 vs X4) from genotypic data (DNA sequence).

**Developed at:** University of Cologne, Germany  
**Published at:** http://coreceptor.geno2pheno.org

### How G2P Works

1. **Input:** V3 loop amino acid sequence (typically 35 amino acids, bounded by cysteines)

2. **Scoring:** Uses a **Position-Specific Scoring Matrix (PSSM)**
   - Each amino acid at each position has a score
   - Scores are based on statistical analysis of thousands of sequences with known tropism
   - Higher scores → more likely to be X4-tropic

3. **Output:** A numerical score that's converted to a **False Positive Rate (FPR)**
   - FPR represents confidence in the prediction
   - **FPR ≥ 3.5%** → classified as R5-tropic
   - **FPR < 3.5%** → classified as X4-tropic

### Why Use This Approach?

**Alternative would be phenotypic assays:** Actually grow the virus in cells with blocked CCR5 or CXCR4 and see what happens.

**Problems with phenotypic testing:**
- Expensive
- Time-consuming (weeks)
- Requires live virus culture (biosafety level 3)
- Technically complex

**Genotypic testing (G2P):**
- Fast (days)
- Cheaper
- Can be done on DNA extracted from blood
- Highly validated and accurate

---

## MiCall's Implementation

### Pipeline Position

MiCall processes tropism testing as the **FIRST step** in the pipeline, before general mapping:

```
FASTQ files → fastq_g2p → V3 mapped reads split off
                        ↓
              g2p.csv + g2p_summary.csv
                        
Unmapped reads → continue to prelim_map → [rest of pipeline]
```

### Step-by-Step Process

#### 1. Read Merging
- Pair-end reads are merged using Gotoh pairwise alignment
- Minimum alignment score required: 20
- If reads don't merge well, they're discarded for G2P (but saved for later steps)

#### 2. Alignment to V3 Reference
- Merged reads are aligned to V3LOOP reference using Gotoh algorithm
- **Reference used:** HIV1-C-BR-JX140663 seed (closest to G2P library reference)
- **Minimum V3 alignment score:** 2× the length of V3 reference
  - This threshold was empirically determined (issue #395)
  - Prevents false positives from non-HIV samples

#### 3. V3 Extraction
- Extract the portion of reads that align to V3LOOP
- Trim to the V3 region boundaries (cysteines at both ends)

#### 4. Quality Filtering

Reads are rejected if they have:
- **Low quality:** >50% of bases are N (ambiguous)
- **Zero length:** empty after trimming
- **Not divisible by 3:** not in reading frame
- **Missing cysteines:** doesn't start with C or end with C
- **Too many ambiguous codons:** >1 ambiguous codon or >2 possible amino acids at one position
- **Stop codons:** premature termination
- **Wrong length:** V3 length must be 32-40 amino acids

#### 5. Translation
- Translate nucleotide sequence to amino acids
- Handle ambiguous codons appropriately

#### 6. G2P Scoring

For each valid sequence:
1. Align to the G2P reference standard: `CTRPNXNNTXXRKSIRIXXXGPGQXXXAFYATXXXXGDIIGDIXXRQAHC`
2. Calculate score using the PSSM matrix
3. Convert score to FPR using empirical lookup table
4. Classify: FPR ≥ 3.5 → R5, FPR < 3.5 → X4

#### 7. Output Generation

**Individual sequence results (g2p.csv):**
- Each unique V3 sequence
- Count of how many reads had this sequence
- G2P score
- FPR value
- Call (R5 or X4)
- Amino acid sequence
- Any errors/warnings

**Sample-level summary (g2p_summary.csv):**
- Total mapped reads
- Total valid reads
- Number of X4 calls
- Percentage of X4
- **Final call:**
  - If ≥2% of valid reads are X4 → sample called X4
  - Otherwise → sample called R5
  - Empty if <7,500 valid reads OR <75% valid

### Key Implementation Details

**Minimum counts:**
- Default: sequences appearing <3 times are excluded
- Can be adjusted via `min_count` parameter

**Validity thresholds:**
- Need ≥7,500 valid reads for a call
- Need ≥75% of mapped reads to be valid
- These ensure sufficient data quality

**Handling minority variants:**
- The pipeline reports ALL sequences above threshold
- The 2% cutoff for X4 classification accounts for minority variants
- This is conservative - ensures maraviroc won't be prescribed if significant X4 population exists

---

## Clinical Workflow at BC CfE

### Sample Arrival
1. Clinician orders V3 tropism test
2. Requisition entered into QAI
3. Blood sample received in lab
4. DNA extracted and V3 region amplified by PCR

### Sequencing
1. V3 amplicons loaded onto MiSeq
2. Paired-end sequencing (~250bp × 2)
3. FASTQ files automatically copied to RAW_DATA drive

### MiCall Processing
1. MiCall watcher detects new run
2. Launches fastq_g2p step via Kive
3. Results written to g2p.csv and g2p_summary.csv

### Reporting
1. Lab staff review g2p_summary.csv
2. Check validity metrics (enough reads? high enough quality?)
3. Report final call to clinician: R5 or X4
4. If R5 → maraviroc can be considered
5. If X4 or insufficient data → maraviroc contraindicated

---

## Why This Matters

### Patient Safety
- Wrong tropism call → prescribing maraviroc to X4 patient → treatment failure
- MiCall's thresholds and quality checks minimize this risk

### Research vs Clinical
- This is MiCall's ONLY routine clinical use at BC CfE
- All other MiCall work is research (proviral analysis, HCV, SARS-CoV-2)
- ReCall handles standard HIV resistance testing for clinical purposes

### Deep Sequencing Advantage
- NGS can detect minority X4 variants
- Sanger might miss a 5% X4 population
- MiCall's 2% threshold catches these

---

## Key Differences from Resistance Testing

| Feature | Tropism (V3) | Resistance |
|---------|--------------|------------|
| Target | V3 loop only | Pol gene (PR, RT, IN) |
| Length | ~105bp (35 aa) | ~3000bp (multiple genes) |
| Algorithm | Geno2pheno PSSM | Stanford HIVdb rules |
| Output | R5 or X4 | Drug resistance levels |
| Clinical use | Before maraviroc | Before most ARTs |
| MiCall role | Primary tool | Research only (clinical uses ReCall) |
| Pipeline step | First (fastq_g2p) | Late (resistance module) |

---

## Technical Implementation Files

### Code Files
- `micall/g2p/fastq_g2p.py` - main implementation
- `micall/g2p/pssm_lib.py` - PSSM scoring and G2P algorithm
- `micall/g2p/g2p.matrix` - scoring matrix data
- `micall/g2p/g2p_fpr.txt` - G2P score to FPR lookup table

### Reference Data
- V3LOOP coordinate reference (amino acid sequence)
- HIV seed references for initial alignment
- G2P PSSM matrix (position-specific scores)

### Documentation
- `docs/design/fastq_g2p.md` - design decisions
- `docs/steps.md` - pipeline flow

---

## Summary

**In plain language:** 

V3 tropism testing tells us if HIV uses CCR5 or CXCR4 to enter cells. This matters because the drug maraviroc only works against CCR5-using virus. 

MiCall sequences the V3 loop region, translates it to amino acids, and feeds it into the geno2pheno algorithm. That algorithm uses a scoring matrix to predict R5 vs X4 based on which amino acids appear at which positions.

The key insight is that V3 sequence determines coreceptor usage, and we can predict this computationally instead of doing expensive phenotypic assays.

This is the ONLY clinical use of MiCall at BC CfE - everything else is research. For resistance testing, the lab uses ReCall (Sanger sequencing) for clinical purposes.
