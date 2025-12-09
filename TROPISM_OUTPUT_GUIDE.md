# Tropism Testing Output Guide

## Overview

This document describes the final products of HIV tropism testing in the MiCall/QAI system, their structure, and how to obtain example outputs.

---

## What Are Tropism Test Outputs?

Tropism testing determines whether HIV uses CCR5 (R5-tropic) or CXCR4 (X4-tropic) coreceptors to enter cells. This is critical for prescribing maraviroc, a CCR5 antagonist drug.

**Key Point**: This is the ONLY clinical application of MiCall at BC CfE (all other uses are research).

---

## Final Output Files

MiCall's `fastq_g2p` step produces two main output files:

### 1. `g2p.csv` - Individual Sequence Results

Contains detailed predictions for each unique V3 sequence variant found in the sample.

**Columns:**
- `rank` - Sequences ranked by count (most common = 1)
- `count` - Number of reads with this exact sequence
- `g2p` - G2P score from PSSM algorithm (higher = more X4-like)
- `fpr` - False Positive Rate (%)
- `call` - R5 or X4 classification (FPR ≥ 3.5% → R5, FPR < 3.5% → X4)
- `seq` - V3 amino acid sequence (without gaps)
- `aligned` - Sequence aligned to G2P reference standard
- `error` - Error message if sequence failed validation
- `comment` - Additional notes (e.g., "ambiguous" for mixed bases)

**Example:**
```csv
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,15234,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
2,823,0.454349,2.6,X4,CMRPNNNTRKSIHIGPGRAFYATGEIIGDIRRAHC,CMRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RRAHC,,
3,45,,,,CTR,,cysteines,
```

**Interpretation:**
- Row 1: Most common sequence (15,234 reads), R5-tropic
- Row 2: Second most common (823 reads), X4-tropic  
- Row 3: Invalid sequence (missing closing cysteine)

### 2. `g2p_summary.csv` - Sample-Level Call

Contains the final clinical call for the entire sample.

**Columns:**
- `mapped` - Total reads that mapped to V3LOOP reference
- `valid` - Number of reads that passed quality filters
- `X4calls` - Number of valid reads classified as X4
- `X4pct` - Percentage of valid reads that are X4
- `final` - Final call: R5 or X4 (or empty if insufficient data)
- `validpct` - Percentage of mapped reads that were valid

**Example:**
```csv
mapped,valid,X4calls,X4pct,final,validpct
16102,16057,823,5.12,X4,99.72
```

**Decision Logic:**
- If ≥2% of valid reads are X4 → sample called **X4**
- Otherwise → sample called **R5**
- **Empty call** if:
  - <7,500 valid reads, OR
  - <75% of mapped reads are valid

**Clinical Significance:**
- **X4** → Maraviroc contraindicated (won't work)
- **R5** → Maraviroc can be considered
- **Empty** → Insufficient data, cannot prescribe maraviroc

---

## Quality Filters Applied

Reads are rejected from g2p.csv (marked with error code) if:

| Error Code | Reason |
|------------|--------|
| `cysteines` | Doesn't start with C or end with C |
| `notdiv3` | Length not divisible by 3 (frameshift) |
| `> 2 ambiguous` | >1 ambiguous codon or >2 amino acids at one position |
| `stop codons` | Contains premature stop codons (*) |
| `length` | V3 length not 32-40 amino acids |
| `failed to align` | Could not align to G2P reference |
| `count < N` | Appeared fewer than min_count times (default: 3) |

---

## Additional Output Files

### 3. `g2p_aligned.csv`

Reads that successfully mapped to V3LOOP, aligned to HIV seed reference (not G2P reference).

**Purpose:** Input for downstream `aln2counts` step to generate coverage plots.

**Format:** Same as general `aligned.csv` files:
```csv
refname,qcut,rank,count,offset,seq
HIV1-C-BR-JX140663-seed,15,0,1234,150,TGTACAAGACCCAACAAC...
```

### 4. `merged_contigs.csv`

Consensus sequences from amplicon reads (based on insert length clustering).

**Purpose:** Used in denovo assembly pipeline.

**Format:**
```csv
contig
TGTACAAGACCCAACAACAATACAAGAAAAAGTATACATATAGGACCAGGGAGAGCATTTTATGCAACAGGAGAAATAATAGGAGATATAAGACAAGCACATTGT
```

### 5. `not_v3_r1.fastq` and `not_v3_r2.fastq`

FASTQ files containing reads that did NOT map to V3LOOP reference.

**Purpose:** These reads continue through the rest of MiCall pipeline (prelim_map, remap, etc.) for other analyses.

---

## Database Storage (QAI)

Tropism results are stored in Oracle database tables:

### `hiv_env_v3_seq`
- Stores individual sequence data (from g2p.csv)
- Primary key: `enum` (specimen number)

### `hiv_env_v3_scores`  
- Stores G2P scores and summary data
- Primary key: `enum` (specimen number)
- Schema: `specimen.hiv_env_v3_scores`

### Access Pattern
Results are accessed through QAI web interface:
- Lab staff review results in the MiCall module
- Navigate to run → sample → view G2P results
- Summary shows final call (R5/X4) and quality metrics

---

## How to Get Example Outputs

### Option 1: Use Test Data from MiCall Repository

The `deps.local/MiCall/micall/tests/test_fastq_g2p.py` file contains example outputs embedded in test cases.

**Look for:**
```python
expected_g2p_csv = """
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,1,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
"""

expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
1,1,0,0.00,R5,100.00
"""
```

### Option 2: Run MiCall on Sample Data

```bash
# Navigate to MiCall directory
cd deps.local/MiCall

# Run fastq_g2p on test FASTQ files
python -m micall.g2p.fastq_g2p \
    path/to/read1.fastq \
    path/to/read2.fastq \
    g2p_output.csv \
    g2p_summary_output.csv \
    unmapped_r1.fastq \
    unmapped_r2.fastq \
    g2p_aligned.csv \
    merged_contigs.csv
```

### Option 3: Access Real Run Data (if you have access)

On the BC CfE network:

1. **Find run data:**
   ```
   RAW_DATA/<run_name>/Results/<sample_name>/
   ```

2. **Look for files:**
   - `g2p.csv`
   - `g2p_summary.csv`
   - `g2p_aligned.csv`

3. **Check MiSeq run folder structure:**
   ```
   <run_name>/
   ├── SampleSheet.csv
   └── Results/
       └── <sample_name>/
           ├── g2p.csv
           ├── g2p_summary.csv
           ├── g2p_aligned.csv
           └── merged_contigs.csv
   ```

### Option 4: Query QAI Database (requires access)

Connect to Oracle database and query:

```sql
-- Get V3 scores for a specimen
SELECT * FROM specimen.hiv_env_v3_scores
WHERE enum = 'CFE_12345_V3';

-- Get individual sequences
SELECT * FROM hiv_env_v3_seq
WHERE enum = 'CFE_12345_V3';
```

---

## Example: Complete Output Set

For specimen **CFE_12345_V3**:

### g2p.csv
```csv
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,8234,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
2,234,0.454349,2.6,X4,CMRPNNNTRKSIHIGPGRAFYATGEIIGDIRRAHC,CMRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RRAHC,,
3,45,0.120345,28.5,R5,CTRPNNNTRKSIHIGPGRAFYATGDIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GDIIGDI--RQAHC,,
4,12,,,,CTRPNNN,,length,
5,8,,,,CTR,,cysteines,
```

### g2p_summary.csv
```csv
mapped,valid,X4calls,X4pct,final,validpct
8533,8513,234,2.75,X4,99.77
```

**Interpretation:**
- 8,533 total reads mapped to V3LOOP
- 8,513 passed quality filters (99.77% validity)
- 234 reads (2.75%) classified as X4
- **Final call: X4** (because 2.75% > 2% threshold)
- **Clinical action:** Maraviroc contraindicated for this patient

---

## Key Thresholds & Constants

| Parameter | Value | Purpose |
|-----------|-------|---------|
| FPR cutoff | 3.5% | FPR ≥ 3.5 → R5, FPR < 3.5 → X4 |
| X4 percentage threshold | 2% | ≥2% X4 reads → sample called X4 |
| Minimum valid reads | 7,500 | Required for confident call |
| Minimum valid percentage | 75% | Required for confident call |
| Default min_count | 3 | Sequences appearing <3 times excluded |
| V3 length range | 32-40 aa | Valid V3 loop length |
| Min V3 alignment score | 2 × len(V3LOOP) | ~210 for typical V3 |

---

## Pipeline Position

```
FASTQ files (paired-end reads)
    ↓
┌───────────────────────────────┐
│  fastq_g2p (FIRST STEP)       │
│  - Merge read pairs           │
│  - Align to V3LOOP reference  │
│  - Extract V3 region          │
│  - Translate to amino acids   │
│  - Run G2P algorithm          │
│  - Generate predictions       │
└───────────────────────────────┘
    ↓                        ↓
g2p.csv                  not_v3_*.fastq
g2p_summary.csv          (continue to prelim_map)
g2p_aligned.csv
merged_contigs.csv
    ↓
Downloaded for review
Uploaded to QAI database
Clinical report generated
```

---

## Reporting to Clinicians

The lab staff reviews the results and reports to clinicians:

**Report Format:**
```
Specimen: CFE_12345_V3
Test: HIV-1 Tropism (V3 Loop Sequencing)

Result: X4-tropic virus detected

Interpretation:
The patient's HIV population uses CXCR4 coreceptor for cell entry.
Maraviroc (CCR5 antagonist) is NOT recommended.

Technical Details:
- Total valid reads: 8,513
- X4 percentage: 2.75%
- Quality: Excellent (99.77% validity)
```

---

## Common Scenarios

### Scenario 1: Pure R5 Population
```csv
mapped,valid,X4calls,X4pct,final,validpct
12000,11850,0,0.00,R5,98.75
```
→ Maraviroc can be considered

### Scenario 2: Mixed Population with Minority X4
```csv
mapped,valid,X4calls,X4pct,final,validpct
15000,14800,400,2.70,X4,98.67
```
→ X4 detected (2.7% > 2%), maraviroc contraindicated

### Scenario 3: Insufficient Data
```csv
mapped,valid,X4calls,X4pct,final,validpct
6000,5800,0,0.00,,96.67
```
→ No call (< 7,500 valid reads), repeat test needed

### Scenario 4: Poor Quality Sample
```csv
mapped,valid,X4calls,X4pct,final,validpct
10000,6000,0,0.00,,60.00
```
→ No call (< 75% validity), sample quality issues

---

## Differences from Resistance Testing

| Feature | Tropism (V3) | Resistance Testing |
|---------|--------------|-------------------|
| Target region | V3 loop (~105 bp) | Pol gene (~3000 bp) |
| Algorithm | Geno2pheno PSSM | Stanford HIVdb |
| Output | R5/X4 call | Drug resistance levels |
| Clinical use | Before maraviroc | Before most ARTs |
| Tool | **MiCall** (clinical) | ReCall (Sanger) |
| Pipeline step | First (fastq_g2p) | Late (resistance module) |
| Thresholds | FPR 3.5%, 2% minority | Various by mutation |

---

## Summary

**Tropism test final products:**
1. **g2p.csv** - Individual sequence predictions
2. **g2p_summary.csv** - Sample-level clinical call
3. Supporting files (aligned reads, contigs)

**To get examples:**
- Extract from MiCall test suite (easiest)
- Run MiCall on sample data
- Access production run folders (requires network access)
- Query QAI database (requires credentials)

**Clinical significance:**
- R5 → Maraviroc possible
- X4 → Maraviroc contraindicated
- Empty → Insufficient data, cannot proceed

**This is MiCall's ONLY clinical application** - all other MiCall analyses are research-only.
