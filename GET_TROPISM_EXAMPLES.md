# Quick Guide: How to Get Tropism Test Examples

## TL;DR

The easiest way to see tropism output examples is to look at the **test cases** in MiCall's test suite.

---

## Method 1: Extract Examples from Test Suite (EASIEST)

### Location
```
deps.local/MiCall/micall/tests/test_fastq_g2p.py
```

### What to Look For

Search for `expected_g2p_csv` and `expected_summary_csv` in the test file. You'll find multiple examples.

### Example 1: Simple R5 Result
```python
# Around line 66-74
expected_g2p_csv = """
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,1,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
"""

expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
1,1,0,0.00,R5,100.00
"""
```

### Example 2: X4 Result (Mixed Population)
```python
# Around line 110-120
expected_g2p_csv = """
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,2,0.454349,2.6,X4,CMRPNNNTRKSIHIGPGRAFYATGEIIGDIRRAHC,CMRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RRAHC,,
2,1,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
"""

expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
3,3,2,66.67,X4,100.00
"""
```

### Example 3: Insufficient Data
```python
# Around line 169-179
expected_g2p_csv = """
rank,count,g2p,fpr,call,seq,aligned,error,comment
1,300,0.067754,42.3,R5,CTRPNNNTRKSIHIGPGRAFYATGEIIGDIRQAHC,CTRPN-NNT--RKSIHI---GPGR---AFYAT----GEIIGDI--RQAHC,,
2,100,,,,CTR,,cysteines,
"""

expected_summary_csv = """
mapped,valid,X4calls,X4pct,final,validpct
400,300,0,0.00,,75.00
"""
# Note: final is empty because min_valid=301 (threshold not met)
```

### Example 4: Invalid Sequences
```python
# Various error conditions shown throughout test file:
# - "cysteines" error (missing start/end C)
# - "notdiv3" error (not divisible by 3)
# - "> 2 ambiguous" error (too many mixed bases)
# - "stop codons" error (premature stop)
# - "length" error (wrong V3 length)
# - "count < 3" (below minimum threshold)
```

---

## Method 2: Run MiCall Locally (MODERATE)

### Prerequisites
```bash
cd deps.local/MiCall
pip install -e .
```

### Create Test FASTQ Files

You'll need paired-end FASTQ files with V3 amplicon sequences. The test suite uses synthetic data, but you could create minimal test files:

**read1.fastq:**
```
@M01234:1:000000000-ABCDE:1:1:12345:1234 1:N:0:1
TGTACAAGACCCAACAACAATACAAGAAAAAGTATACATATAGGACCAGGGAGAGCATTTTATGCAACAGGAGAAATAATAGGAGATATAAGACAAGCACATTGT
+
GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
```

**read2.fastq:**
```
@M01234:1:000000000-ABCDE:1:1:12345:1234 2:N:0:1
ACAATGTGCTGCTTCTTATATCTCCTATTATTTCTCCTGTTGCATAAAATGCTCTCCCTGGTCCTATATGTATACTTTTTCTTGTATTGTTGTTGGGTCTTGTACA
+
GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
```

### Run the Command
```bash
python -m micall.g2p.fastq_g2p \
    read1.fastq \
    read2.fastq \
    g2p_output.csv \
    g2p_summary_output.csv \
    unmapped_r1.fastq \
    unmapped_r2.fastq \
    g2p_aligned.csv \
    merged_contigs.csv
```

### Check Outputs
```bash
cat g2p_output.csv
cat g2p_summary_output.csv
```

---

## Method 3: Access Production Data (REQUIRES ACCESS)

### If You Have Network Access to BC CfE RAW_DATA

1. **Find a recent V3 run:**
   ```
   /path/to/RAW_DATA/*/Results/
   ```

2. **Look for samples with V3 in the name:**
   ```bash
   find /path/to/RAW_DATA -name "*V3*" -type d
   ```

3. **Check for output files:**
   ```bash
   ls -la /path/to/RAW_DATA/<run_name>/Results/<sample_V3>/g2p*.csv
   ```

4. **Copy examples:**
   ```bash
   cp /path/to/RAW_DATA/<run_name>/Results/<sample_V3>/g2p.csv ./example_g2p.csv
   cp /path/to/RAW_DATA/<run_name>/Results/<sample_V3>/g2p_summary.csv ./example_g2p_summary.csv
   ```

### Typical Run Folder Structure
```
20-DEC-2024.M12345/
├── SampleSheet.csv
├── InterOp/
└── Results/
    ├── CFE_12345_V3/
    │   ├── g2p.csv
    │   ├── g2p_summary.csv
    │   ├── g2p_aligned.csv
    │   ├── merged_contigs.csv
    │   ├── not_v3_r1.fastq
    │   └── not_v3_r2.fastq
    └── CFE_67890_V3/
        └── ...
```

---

## Method 4: Query QAI Database (REQUIRES CREDENTIALS)

### Connect to Oracle Database
```ruby
# From QAI Rails console
rails console

# Query V3 scores
scores = HivEnvV3Score.where("enum LIKE ?", "%V3")
scores.first.inspect

# Query V3 sequences  
seqs = HivEnvV3Seq.where("enum LIKE ?", "%V3")
seqs.first.inspect
```

### SQL Query
```sql
-- Get recent V3 test results
SELECT * FROM specimen.hiv_env_v3_scores
WHERE enum LIKE '%_V3'
ORDER BY enum DESC
FETCH FIRST 10 ROWS ONLY;

-- Get sequence details
SELECT * FROM hiv_env_v3_seq
WHERE enum = 'CFE_12345_V3';
```

---

## Method 5: Generate from Test Execution

### Run MiCall's Test Suite
```bash
cd deps.local/MiCall

# Run specific G2P tests
python -m pytest micall/tests/test_fastq_g2p.py -v

# Run with output capture
python -m pytest micall/tests/test_fastq_g2p.py::WriteRowsTest::testSummaryX4 -s
```

The tests create temporary files that you can inspect during debugging.

---

## Recommended Approach

**For presentation purposes:**
1. Use **Method 1** (extract from test suite) - immediate, no setup needed
2. Create a simple table showing the key examples
3. Highlight the difference between R5 and X4 calls

**For hands-on learning:**
1. Start with **Method 1** to understand the format
2. Try **Method 2** to see the pipeline in action
3. If available, use **Method 3** to see real clinical data

**For development:**
1. Use **Method 1** for unit test expectations
2. Use **Method 2** for integration testing
3. Use **Method 4** for validating production data

---

## Quick Reference: Key Examples

### Pure R5 Sample (Maraviroc OK)
```csv
# g2p_summary.csv
mapped,valid,X4calls,X4pct,final,validpct
12000,11850,0,0.00,R5,98.75
```

### X4 Detected (Maraviroc Contraindicated)
```csv
# g2p_summary.csv
mapped,valid,X4calls,X4pct,final,validpct
15000,14800,400,2.70,X4,98.67
```

### Insufficient Data (No Call)
```csv
# g2p_summary.csv
mapped,valid,X4calls,X4pct,final,validpct
6000,5800,0,0.00,,96.67
```

---

## Next Steps

1. **Open** `deps.local/MiCall/micall/tests/test_fastq_g2p.py`
2. **Search** for `expected_g2p_csv`
3. **Copy** examples you need
4. **Understand** the column meanings (see TROPISM_OUTPUT_GUIDE.md)
5. **Adapt** for your presentation needs

That's it! You now have multiple ways to get tropism test examples.
