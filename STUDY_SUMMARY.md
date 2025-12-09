# Study Summary: MiCall & QAI Tropism Testing

## Key Findings

### What is Tropism Testing?

HIV tropism testing determines which **coreceptor** the virus uses to enter cells:
- **CCR5** → R5-tropic virus
- **CXCR4** → X4-tropic virus

**Clinical Importance**: Maraviroc (CCR5 antagonist drug) ONLY works on R5-tropic virus. If the patient has X4-tropic virus, maraviroc is useless.

### MiCall's Role

**Critical Context**: Tropism testing is the **ONLY clinical application** of MiCall at BC CfE.
- All other MiCall work = research
- Clinical resistance testing = done by ReCall (Sanger sequencing)

### The Technology

**Genotypic Approach**: Sequence the V3 loop of HIV envelope gene → predict tropism
- Alternative would be phenotypic assay (expensive, slow, requires live virus)
- V3 loop amino acid sequence determines coreceptor usage
- **Geno2pheno algorithm**: Uses PSSM (Position-Specific Scoring Matrix) to predict R5 vs X4

---

## Final Products of Tropism Tests

### Two Main Output Files

#### 1. `g2p.csv` - Individual Sequence Results

Lists every unique V3 sequence variant found, with predictions:

**Structure:**
| Column | Description |
|--------|-------------|
| rank | Sequences ranked by abundance |
| count | Number of reads with this sequence |
| g2p | G2P PSSM score |
| fpr | False Positive Rate (%) |
| call | R5 or X4 |
| seq | V3 amino acid sequence |
| aligned | Aligned to G2P reference |
| error | Error message if invalid |
| comment | Notes (e.g., "ambiguous") |

**Example:**
```csv
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,8234,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
2,234,0.454349,2.6,X4,CMRPNNNTRKSIHIGPGRAFYATGEIIGDIRRAHC,CMRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RRAHC,,
3,12,,,,CTRPNNN,,length,
```

#### 2. `g2p_summary.csv` - Clinical Call

Single row with sample-level decision:

**Structure:**
| Column | Description |
|--------|-------------|
| mapped | Total reads mapped to V3LOOP |
| valid | Reads passing quality filters |
| X4calls | Number of X4 reads |
| X4pct | Percentage X4 |
| final | **R5, X4, or empty** |
| validpct | % of mapped that are valid |

**Example:**
```csv
mapped,valid,X4calls,X4pct,final,validpct
8533,8513,234,2.75,X4,99.77
```

**Decision Logic:**
- `final = X4` if ≥2% of valid reads are X4
- `final = R5` if <2% of valid reads are X4
- `final = empty` if insufficient data (<7,500 valid reads OR <75% validity)

**Clinical Action:**
- **X4** → Do NOT prescribe maraviroc
- **R5** → Maraviroc can be considered
- **Empty** → Insufficient data, repeat test

---

## How to Get Examples

### Best Option: Extract from MiCall Test Suite

**File**: `deps.local/MiCall/micall/tests/test_fastq_g2p.py`

**What to do**: Search for `expected_g2p_csv` and `expected_summary_csv`

**You'll find examples like:**

```python
# Pure R5 result
expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
1,1,0,0.00,R5,100.00
"""

# X4 detected (66% of reads are X4)
expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
3,3,2,66.67,X4,100.00
"""

# Insufficient data (no call)
expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
400,300,0,0.00,,75.00
"""
```

**Why this is best:**
- No setup required
- Multiple scenarios documented
- Shows expected format exactly
- Includes edge cases and errors

### Alternative: Run MiCall Pipeline

```bash
cd deps.local/MiCall
python -m micall.g2p.fastq_g2p \
    read1.fastq read2.fastq \
    g2p.csv g2p_summary.csv \
    unmapped1.fq unmapped2.fq \
    aligned.csv contigs.csv
```

### Alternative: Access Production Data (requires network access)

```
/path/to/RAW_DATA/<run_name>/Results/<sample_V3>/
├── g2p.csv
├── g2p_summary.csv
├── g2p_aligned.csv
└── merged_contigs.csv
```

---

## Pipeline Flow

```
FASTQ files (paired-end reads from MiSeq)
    ↓
┌─────────────────────────────────────┐
│ fastq_g2p (FIRST PIPELINE STEP)    │
│                                     │
│ 1. Merge read pairs                │
│ 2. Align to V3LOOP reference       │
│ 3. Extract V3 region                │
│ 4. Quality filters                  │
│ 5. Translate to amino acids        │
│ 6. Run Geno2pheno algorithm        │
│ 7. Generate predictions             │
└─────────────────────────────────────┘
    ↓
┌─────────────────┬───────────────────┐
│                 │                   │
│  V3 Results     │  Non-V3 Reads     │
│                 │                   │
├─────────────────┤                   │
│ g2p.csv         │  not_v3_r1.fastq  │
│ g2p_summary.csv │  not_v3_r2.fastq  │
│ g2p_aligned.csv │                   │
│ merged_contigs  │                   │
└─────────────────┘                   │
    ↓                                  ↓
Downloaded for review        Continue to prelim_map
Uploaded to QAI              (rest of MiCall pipeline)
Clinical report generated    (for other analyses)
```

---

## Quality Control

### Reads Rejected If:

| Error | Condition |
|-------|-----------|
| cysteines | Doesn't start with C or end with C |
| notdiv3 | Length not divisible by 3 |
| > 2 ambiguous | Too many mixed bases |
| stop codons | Contains premature stops |
| length | V3 not 32-40 amino acids |
| count < N | Below minimum count threshold |

### Sample-Level Quality Thresholds:

- **Minimum valid reads**: 7,500
- **Minimum validity**: 75% (valid/mapped)
- **X4 percentage threshold**: 2%
- **FPR cutoff**: 3.5% (≥3.5 → R5, <3.5 → X4)

---

## QAI Integration

### Database Tables

**`hiv_env_v3_seq`**
- Stores individual sequence data
- Primary key: `enum` (specimen number)

**`hiv_env_v3_scores`**
- Stores G2P scores and summary
- Schema: `specimen.hiv_env_v3_scores`
- Primary key: `enum`

### Web Interface

Lab staff access results through QAI:
1. Navigate to MiCall module
2. Select run → sample
3. View G2P results
4. Review final call (R5/X4)
5. Check quality metrics

---

## Real-World Example

### Scenario: Patient Needs Salvage Therapy

**Clinical Question**: Can we use maraviroc?

**Test Ordered**: HIV-1 V3 Tropism

**Sample Processing**:
1. Blood draw
2. DNA extraction
3. V3 PCR amplification
4. MiSeq sequencing
5. MiCall analysis

**Results**:
```csv
# g2p_summary.csv
mapped,valid,X4calls,X4pct,final,validpct
15234,15189,412,2.71,X4,99.70
```

**Interpretation**:
- 15,234 reads mapped to V3
- 15,189 passed quality filters (99.7% validity = excellent)
- 412 reads (2.71%) are X4-tropic
- **Final call: X4**

**Clinical Action**:
- Report to clinician: "X4-tropic virus detected"
- Maraviroc is **contraindicated**
- Consider alternative salvage drugs

**Why the 2% threshold matters**:
- Deep sequencing detects minority variants
- Even 2-3% X4 population can lead to treatment failure
- Sanger sequencing might miss these minorities
- Conservative approach protects patient safety

---

## Key Takeaways

1. **Purpose**: Tropism testing determines if maraviroc will work
2. **Method**: Sequence V3 loop → predict R5/X4 with Geno2pheno
3. **Outputs**: 
   - `g2p.csv` - individual sequences
   - `g2p_summary.csv` - clinical call
4. **Threshold**: ≥2% X4 reads → sample called X4
5. **Clinical use**: This is MiCall's ONLY clinical application
6. **Examples**: Found in `test_fastq_g2p.py`

---

## Additional Output Files

Beyond the two main files:

### `g2p_aligned.csv`
- Reads aligned to HIV seed reference
- Used for downstream coverage plots
- Input to `aln2counts` step

### `merged_contigs.csv`
- Consensus sequences from amplicons
- Based on insert length clustering
- Used in denovo assembly

### `not_v3_r1.fastq` / `not_v3_r2.fastq`
- Reads that didn't map to V3
- Continue through rest of pipeline
- For other analyses (resistance, etc.)

---

## Comparison: Tropism vs Resistance

| Feature | Tropism | Resistance |
|---------|---------|------------|
| Target | V3 loop (~105 bp) | Pol gene (~3000 bp) |
| Algorithm | Geno2pheno | Stanford HIVdb |
| Output | R5 or X4 | Drug resistance levels |
| Pipeline | MiCall | ReCall (Sanger) |
| Clinical use | Before maraviroc | Before most ARTs |
| When | First step | Late step |
| Threshold | 2% minority | Varies by mutation |

---

## References

### Documentation
- `TROPISM_TESTING_REPORT.md` - Detailed technical background
- `TROPISM_OUTPUT_GUIDE.md` - Comprehensive output reference
- `GET_TROPISM_EXAMPLES.md` - Quick guide to finding examples
- `deps.local/MiCall/docs/steps.md` - Pipeline steps
- `deps.local/MiCall/micall/tests/test_fastq_g2p.py` - Test cases with examples

### Code
- `deps.local/MiCall/micall/g2p/fastq_g2p.py` - Main implementation
- `deps.local/MiCall/micall/g2p/pssm_lib.py` - G2P algorithm
- `deps.local/MiCall/micall/g2p/g2p.matrix` - PSSM matrix data
- `deps.local/MiCall/micall/g2p/g2p_fpr.txt` - Score to FPR lookup

### QAI
- `deps.local/QAI/models/hiv_env_v3_seq.rb` - Sequence model
- `deps.local/QAI/models/hiv_env_v3_scores.rb` - Scores model
- `deps.local/QAI/controllers/qcs_micall_controller.rb` - Web interface

---

## For Your Presentation

**Key Points to Emphasize:**

1. **Clinical Significance**
   - Only clinical use of MiCall
   - Determines drug choice
   - Protects patient safety

2. **Technology Advantage**
   - Deep sequencing detects minorities
   - Faster than phenotypic assays
   - More accurate than Sanger

3. **Output Format**
   - Two simple CSV files
   - Clear R5/X4 call
   - Quality metrics included

4. **Real-World Impact**
   - Guides salvage therapy
   - Prevents treatment failures
   - Enables precision medicine

**Where to Get Examples:**
→ `deps.local/MiCall/micall/tests/test_fastq_g2p.py`
→ Search for `expected_g2p_csv`
→ Multiple scenarios documented

**Visual Aids:**
- Show example g2p.csv with highlighted columns
- Show g2p_summary.csv with decision logic
- Pipeline flowchart (V3 split from main pipeline)
- Clinical decision tree (R5 → consider maraviroc, X4 → don't use)

---

## Questions Answered

✅ **How do final products of tropism tests look?**
→ Two CSV files: `g2p.csv` (details) + `g2p_summary.csv` (clinical call)

✅ **How to get an example?**
→ Extract from `test_fastq_g2p.py` test cases (easiest method)

✅ **What do MiCall and QAI do?**
→ MiCall processes sequences → QAI stores/displays results

✅ **Where are results stored?**
→ Database: `hiv_env_v3_seq` + `hiv_env_v3_scores` tables

✅ **How is this different from resistance testing?**
→ Different target region, algorithm, and clinical application

---

**You're now ready to explain tropism testing outputs!** 🎉
