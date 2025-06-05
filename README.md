# Genomic Variant Pipeline with ADAM and PySpark

This project processes genomic alignment and variant files (SAM/BAM/VCF) into ADAM format using Apache ADAM and performs variant filtering using Apache Spark.

## Requirements

- AWS S3
- Amazon EMR (with ADAM and PySpark installed)
- ADAM CLI (`adam-submit`)
- Python 3

## Usage

```bash
python run_pipeline.py s3://your-bucket/input s3://your-bucket/output
