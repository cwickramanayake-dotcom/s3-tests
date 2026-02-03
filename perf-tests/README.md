# S3 Performance Testing Framework

## 📝 Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `--target <name>` | Target configuration (can repeat) | Required |
| `--duration <time>` | Duration per test (e.g., 3m, 5m) | 3m |
| `--warmup <time>` | Warmup period | 30s |
| `--sizes <list>` | Object sizes (comma-separated) | 4KiB,64KiB,1MiB,16MiB,128MiB |
| `--concurrency <list>` | Concurrency levels | 1,8,32,128 |
| `--iterations <n>` | Iterations per test | 3 |
| `--compare` | Generate comparison dataset | false |
| `--report` | Generate HTML report | false |
| `--use-latest` | Use existing results | false |
| `--skip-cleanup` | Keep temporary objects | false |
| `--verbose` | Enable verbose logging | false |


## ⚡ Quick Start

### 1. Install Prerequisites

**macOS:**
```bash
# Install WARP and jq
brew install minio/stable/warp jq

# Install Python packages
pip3 install pandas matplotlib seaborn jinja2
```

**Linux (Ubuntu/Debian):**
```bash
# Install WARP
wget https://github.com/minio/warp/releases/latest/download/warp_linux_amd64 -O warp
chmod +x warp && sudo mv warp /usr/local/bin/

# Install jq and Python
sudo apt-get update && sudo apt-get install -y jq python3-pip

# Install Python packages
pip3 install pandas matplotlib seaborn jinja2
```

### 2. config Setup

```bash
# S3-Compatible Storage Configuration
# This is the comparison target against AWS S3 baseline

# S3 Endpoint
# Example: https://minio.example.com

S3_ENDPOINT="https://your-s3-compatible-endpoint.example.com"

# Access Key ID
S3_ACCESS_KEY="YOUR_ACCESS_KEY_ID"

# Secret Access Key
S3_SECRET_KEY="YOUR_SECRET_ACCESS_KEY"

# Bucket name (must exist, will NOT be created automatically)
S3_BUCKET="warp-benchmark-test"

# Region (if applicable, use same as AWS for fair comparison)
# Some S3-compatible systems don't use regions - set to "us-east-1" or leave empty
S3_REGION="us-east-1"

# Enable TLS (HTTPS)
# Options: true, false
# Recommendation: Use true for production testing, may use false for local/dev
S3_TLS="true"

cd perf-tests
./validate_setup.sh
```

### 3. Configure Targets

Edit configuration files with your S3 credentials:

```bash
# Configure AWS S3
vi targets/aws.env

# Configure S3-compatible storage
vi targets/other.env
```

**Important:** Create the S3 buckets before running benchmarks!

### 4. Run Benchmarks

**Single target (no comparison):**
```bash
# Run benchmarks on AWS S3 only
./warp_s3_benchmark.sh --target aws

# Run benchmarks on other target only
./warp_s3_benchmark.sh --target other --duration 1m
```

**Multiple targets with comparison:**
```bash
# Run both targets and generate comparison report
./warp_s3_benchmark.sh --target aws --target other --compare --report
```

### 5. Run Tests and Generate Report

**Quick test (recommended for first time):**
```bash
# Run quick benchmark suite with automatic report generation
./quick_run_and_report.sh aws
```

**Full benchmark suite:**
```bash
# Run comprehensive benchmark with report
./run_and_report.sh aws

# Run both targets and compare
./run_and_report.sh aws other
```

### 6. View Results

```bash
# Find and open the latest report
open $(find results -name "report.html" -type f | sort | tail -1)  # macOS
xdg-open $(find results -name "report.html" -type f | sort | tail -1)  # Linux
```

## 📊 What Gets Tested

### Default Test Matrix

- **Object Sizes:** 4KiB, 64KiB, 1MiB, 16MiB, 128MiB, 750MiB
- **Concurrency Levels:** 1, 8, 32, 128
- **Operations:** PUT, GET, DELETE, LIST, MIXED
- **Iterations:** 3 per test (median used)
- **Duration:** 3 minutes per test
- **Warmup:** 30 seconds before each test

**Default object sizes:** 4KiB, 64KiB, 1MiB, 16MiB, 128MiB, 750MiB

**Total tests:** 6 sizes × 4 concurrency × 5 operations × 3 iterations = **360 benchmarks**

### Metrics Collected

- **Throughput:** MiB/s (higher is better)
- **Operations/sec:** Request rate (higher is better)
- **Latency:**
  - Average latency
  - P50 (median)
  - P90 (90th percentile)
  - P99 (99th percentile - tail latency)
- **Error Rate:** Percentage of failed operations (lower is better)

## 📈 Output & Reports

### Structured Data

All results are saved as JSON for reproducibility:

- **`summary.json`** - Parsed metrics for each test
- **`merged_summary.json`** - Combined data from multiple targets
- **`metadata.json`** - Environment and configuration details

### Visual Reports

Automatically generated graphs:

- Throughput vs object size
- Throughput vs concurrency
- P99 latency comparisons
- Per-operation performance charts


## � Managing Results

### Directory Structure

Each benchmark run creates a timestamped directory:

```
results/
├── aws/
│   ├── 2024-01-26_10-00-00/     # First run
│   │   ├── raw/
│   │   ├── summary.json
│   │   └── metadata.json
│   └── 2024-01-26_14-30-00/     # Second run
│       ├── raw/
│       ├── summary.json
│       └── metadata.json
├── other/
│   ├── 2024-01-26_11-00-00/
│   └── 2024-01-26_15-00-00/
└── comparison_2024-01-26_15-30-00/    # Comparison report
    ├── merged_summary.json
    ├── charts/
    ├── report.html
    └── metadata.json
```


### Common Workflows

**Workflow 1: Baseline then Compare**
```bash
# Day 1: Establish AWS baseline
./warp_s3_benchmark.sh --target aws

# Day 2: Test alternative storage
./warp_s3_benchmark.sh --target other

# Day 2: Generate comparison report
./warp_s3_benchmark.sh --target aws --target other --use-latest --compare --report
```
### Cleaning Up Old Results

```bash
# Delete results older than 30 days
find results -type d -name "20*" -mtime +30 -exec rm -rf {} \;

# Keep only last 5 benchmark runs per target
cd results/aws && ls -t | tail -n +6 | xargs rm -rf

# Archive old results
tar -czf benchmark-archive-$(date +%Y%m%d).tar.gz results/
```

## �🔧 Usage Examples

### Example 1: Quick Validation

Test a single target quickly:
```bash
./warp_s3_benchmark.sh --target aws --duration 1m --sizes "1MiB" --concurrency "8"
```

### Example 2: Full Comparison

Compare AWS S3 vs S3-compatible storage:
```bash
./warp_s3_benchmark.sh \
  --target aws \
  --target other \
  --compare \
  --report
```

### Example 3: Custom Workload

Test specific scenario matching your production workload:
```bash
./warp_s3_benchmark.sh \
  --target aws \
  --target minio \
  --sizes "1MiB,16MiB" \
  --concurrency "8,32,128" \
  --duration 10m \
  --iterations 5 \
  --compare \
  --report
```

### Example 4: Large Object Testing

Focus on large files (multipart uploads):
```bash
./warp_s3_benchmark.sh \
  --target aws \
  --target other \
  --sizes "128MiB,256MiB,512MiB" \
  --concurrency "8,32" \
  --duration 5m \
  --compare \
  --report
```

### Example 5: Regenerate Report

Use existing data to regenerate report:
```bash
./warp_s3_benchmark.sh \
  --target aws \
  --target other \
  --use-latest \
  --compare \
  --report
```

### Example 6: Run Single Target (Preserve Old Comparisons)

Run benchmarks on one target without affecting old comparison reports:
```bash
# Update only AWS results
./warp_s3_benchmark.sh --target aws

# Old comparison reports are preserved in results/comparison_*/
# Each run creates a new timestamped directory
```

### Example 7: Compare Old Results with New Run

Run one target, then use old and new results for comparison:
```bash
# 1. Run AWS benchmarks (creates results/aws/2024-01-26_10-00-00/)
./warp_s3_benchmark.sh --target aws

# 2. Later, run other target
./warp_s3_benchmark.sh --target other

# 3. Generate comparison from latest results of both targets
./warp_s3_benchmark.sh --target aws --target other --use-latest --compare --report
```

## 🧪 Test Scenarios

### Scenario 1: Migration Validation
**Goal:** Verify acceptable performance

```bash
./warp_s3_benchmark.sh --target aws --target new-storage --compare --report
```

**Look for:** < 25% performance degradation on critical operations

### Scenario 2: Scalability Testing
**Goal:** Understand how performance scales with load

```bash
./warp_s3_benchmark.sh \
  --target production \
  --sizes "1MiB" \
  --concurrency "1,2,4,8,16,32,64,128" \
  --report
```

**Look for:** Linear throughput scaling up to saturation point

### Scenario 3: Workload Simulation
**Goal:** Test with production-like object sizes

```bash
# Analyze your production workload first
# Then test those specific sizes
./warp_s3_benchmark.sh \
  --target aws \
  --target candidate \
  --sizes "512KiB,2MiB,8MiB" \
  --concurrency "16,32" \
  --duration 10m \
  --iterations 5 \
  --compare \
  --report
```

### Scenario 4: Regression Testing
**Goal:** Track performance changes over time

```bash
# Run regularly (e.g., weekly) with consistent parameters
./warp_s3_benchmark.sh \
  --target production \
  --duration 5m \
  --iterations 5 \
  --report

# Compare results/ directories over time
```

## 🔍 Interpreting Results

### Performance Deltas

- **< ±10%:** Within normal variance (not significant)
- **±10-25%:** Noticeable difference (investigate further)
- **> ±25%:** Significant difference (actionable)

### Red Flags

- **High error rates (> 1%):** Configuration or connectivity issues
- **Inconsistent results across iterations:** Network instability
- **Very high P99 latency:** Potential queuing or resource contention
- **Throughput regression > 50%:** Investigate immediately

## 🛠️ Troubleshooting

### Common Issues

**Issue:** `warp: command not found`  
**Fix:** Install WARP (see Quick Start section)

**Issue:** `bucket does not exist`  
**Fix:** Create buckets manually before running benchmarks
```bash
aws s3 mb s3://your-bucket-name --region us-west-2
```

**Issue:** `access denied`  
**Fix:** Verify credentials and IAM permissions (PutObject, GetObject, DeleteObject, ListBucket)

**Issue:** `connection timeout`  
**Fix:** Check endpoint URL, TLS setting, and network connectivity

**Issue:** No graphs in report  
**Fix:** Install Python packages: `pip3 install pandas matplotlib seaborn jinja2`

**Issue:** Inconsistent results  
**Fix:** Increase iterations (`--iterations 5`) and duration (`--duration 5m`)

### Debug Mode

Run with verbose logging:
```bash
./warp_s3_benchmark.sh --target aws --verbose
```