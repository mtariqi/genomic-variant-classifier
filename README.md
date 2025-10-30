# Genomic Variant Pipeline with ADAM and PySpark

A scalable, production-ready genomic data processing pipeline that converts alignment and variant files (SAM/BAM/VCF) into ADAM/Parquet format and performs distributed variant analysis using Apache ADAM and Apache Spark on AWS EMR.

## 🧬 Features

- **Format Conversion**: Transform SAM/BAM/VCF files to efficient Parquet-backed ADAM format
- **Distributed Processing**: Leverage Apache Spark for processing large-scale genomic datasets
- **Variant Filtering**: Filter variants by quality score, depth, and genomic regions
- **AWS Integration**: Seamless S3 storage and EMR cluster processing
- **Reproducible Workflows**: Docker containers and environment specifications
- **Quality Metrics**: Generate comprehensive QC reports with visualizations

## 📋 Requirements

### Infrastructure
- **AWS Account** with S3 and EMR access
- **Amazon EMR Cluster** (EMR 6.x or higher)
  - Spark 3.x
  - ADAM 0.36.0+
  - Python 3.8+

### Software Dependencies
```bash
# Core dependencies
apache-spark >= 3.2.0
pyspark >= 3.2.0
bdg-formats >= 0.16.0
adam-core >= 0.36.0

# Python packages
boto3 >= 1.26.0
pandas >= 1.5.0
numpy >= 1.23.0
matplotlib >= 3.6.0
seaborn >= 0.12.0
```

## 🚀 Installation

### 1. EMR Cluster Setup

Create an EMR cluster with ADAM bootstrap action:

```bash
aws emr create-cluster \
  --name "Genomic-Variant-Pipeline" \
  --release-label emr-6.10.0 \
  --applications Name=Hadoop Name=Spark Name=Hive \
  --ec2-attributes KeyName=your-key-pair \
  --instance-type m5.xlarge \
  --instance-count 3 \
  --bootstrap-actions \
    Path=s3://your-bucket/bootstrap-adam.sh \
  --use-default-roles
```

### 2. Bootstrap Script for ADAM

Create `bootstrap-adam.sh`:

```bash
#!/bin/bash
# Download and install ADAM
wget https://repo1.maven.org/maven2/org/bdgenomics/adam/adam-distribution-spark3_2.12/0.36.0/adam-distribution-spark3_2.12-0.36.0-bin.tar.gz
tar xvfz adam-distribution-spark3_2.12-0.36.0-bin.tar.gz
sudo mv adam-distribution-spark3_2.12-0.36.0 /opt/adam
echo 'export PATH=/opt/adam/bin:$PATH' | sudo tee -a /etc/profile.d/adam.sh
```

### 3. Install Python Dependencies

```bash
pip install -r requirements.txt
```

Complete Project Structure:

```
genomic-variant-pipeline/
├── README.md                          ✅ (created earlier)
├── LICENSE                            ✅ NEW
├── setup.py                           ✅ NEW
├── requirements.txt                   ✅ (created earlier)
├── MANIFEST.in                        ✅ NEW
├── .gitignore                         ✅ NEW
│
├── config/
│   └── pipeline_config.yaml          ✅ NEW
│
├── scripts/
│   ├── convert_to_adam.py            ✅ (created earlier)
│   ├── variant_filtering.py          ✅ (created earlier)
│   ├── quality_control.py            ✅ (created earlier)
│   └── run_pipeline.py               ✅ (created earlier)
│
├── src/
│   ├── __init__.py                   ✅ NEW
│   ├── adam_converter.py             ✅ NEW
│   ├── variant_processor.py          ✅ NEW
│   ├── s3_handler.py                 ✅ NEW
│   └── utils.py                      ✅ NEW
│
├── tests/
│   └── test_pipeline.py              ✅ NEW
│
└── docker/
    └── Dockerfile                    ✅ (created earlier)
```

## 💻 Usage

### Quick Start

```bash
# 1. Upload input files to S3
aws s3 cp sample.bam s3://your-bucket/input/
aws s3 cp reference.fa s3://your-bucket/reference/

# 2. Run the complete pipeline
python scripts/run_pipeline.py \
  --input s3://your-bucket/input/sample.bam \
  --output s3://your-bucket/output/ \
  --reference s3://your-bucket/reference/reference.fa \
  --quality-threshold 30 \
  --depth-threshold 10 \
  --file-type bam
```
### How to Install and Use:

# 1. Clone/create the repository
git clone https://github.com/mtariqi/genomic-variant-pipeline.git
cd genomic-variant-pipeline

# 2. Install in development mode
pip install -e .

# 3. Or install from PyPI (after publishing)
pip install genomic-variant-pipeline

# 4. Use command-line tools
genomic-pipeline --input sample.bam --output results/ --file-type bam

# 5. Or use as Python library
from src import ADAMConverter, VariantProcessor
converter = ADAMConverter()
converter.convert_alignment('sample.bam', 'output.adam')

### Installation Commands:
# Install all dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -e ".[dev]"

# Run tests
pytest tests/

# Build package
python setup.py sdist bdist_wheel

# Install locally
pip install -e .



### Step-by-Step Execution

#### 1. Convert BAM/VCF to ADAM Format

```bash
python scripts/convert_to_adam.py \
  --input s3://your-bucket/input/sample.bam \
  --output s3://your-bucket/adam/sample/ \
  --file-type bam
```

#### 2. Filter Variants

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  scripts/variant_filtering.py \
  --input s3://your-bucket/adam/sample/ \
  --output s3://your-bucket/filtered/ \
  --quality 30 \
  --depth 10
```

#### 3. Filter by Genomic Region (Optional)

```bash
spark-submit \
  --master yarn \
  scripts/variant_filtering.py \
  --input s3://your-bucket/adam/sample/ \
  --output s3://your-bucket/filtered_region/ \
  --quality 30 \
  --depth 10 \
  --chromosome chr1 \
  --start 1000000 \
  --end 2000000
```

#### 4. Generate QC Report

```bash
python scripts/quality_control.py \
  --input s3://your-bucket/filtered/ \
  --output s3://your-bucket/qc_report/
```

## 📊 Example Output

```
=== Variant Statistics ===

Variants per chromosome:
+-----------+-------+
|contigName | count |
+-----------+-------+
|chr1       | 45230 |
|chr2       | 38421 |
|chr3       | 32105 |
...

Variants by type:
+-------------+-------+
|variant_type | count |
+-------------+-------+
|SNP          | 125430|
|INSERTION    | 8234  |
|DELETION     | 7891  |
|COMPLEX      | 1245  |
+-------------+-------+

Quality score statistics:
+-------+------------------+
|summary|variantQuality    |
+-------+------------------+
|count  |142800            |
|mean   |45.67             |
|stddev |12.34             |
|min    |30.0              |
|max    |99.9              |
+-------+------------------+
```

## 🧪 Testing

```bash
# Run unit tests
pytest tests/

# Test with sample data
python scripts/run_pipeline.py \
  --input s3://genomics-test-data/sample.bam \
  --output s3://your-bucket/test-output/ \
  --quality-threshold 20 \
  --depth-threshold 5 \
  --file-type bam
```

## 🐳 Docker Support

Build and run using Docker:

```bash
# Build image
docker build -t genomic-variant-pipeline -f docker/Dockerfile .

# Run container
docker run -it \
  -v ~/.aws:/root/.aws \
  -e AWS_PROFILE=default \
  genomic-variant-pipeline
```

## 📈 Performance Optimization

### EMR Cluster Sizing
- **Small datasets (< 100GB)**: m5.xlarge, 3-5 nodes
- **Medium datasets (100GB - 1TB)**: m5.2xlarge, 5-10 nodes
- **Large datasets (> 1TB)**: m5.4xlarge, 10-20 nodes

### Spark Configuration

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 8g \
  --executor-memory 16g \
  --executor-cores 4 \
  --num-executors 10 \
  --conf spark.default.parallelism=200 \
  --conf spark.sql.shuffle.partitions=200 \
  scripts/variant_filtering.py
```

## 🔧 Configuration

Create `config/pipeline_config.yaml`:

```yaml
quality_thresholds:
  min_quality_score: 30
  min_read_depth: 10
  min_genotype_quality: 20

filtering:
  remove_duplicates: true
  filter_multiallelic: true
  max_missing_rate: 0.1

output:
  format: parquet
  compression: snappy
  partition_by: chromosome

spark:
  executor_memory: "16g"
  executor_cores: 4
  num_executors: 10
```

## 📚 Documentation

For detailed documentation, see:
- [ADAM Documentation](https://adam.readthedocs.io/)
- [PySpark API](https://spark.apache.org/docs/latest/api/python/)
- [AWS EMR Guide](https://docs.aws.amazon.com/emr/)

## 🐛 Troubleshooting

### Common Issues

**Issue**: Out of memory errors
```bash
# Solution: Increase executor memory
--executor-memory 32g --driver-memory 16g
```

**Issue**: S3 access denied
```bash
# Solution: Check IAM roles and permissions
aws s3 ls s3://your-bucket/ --profile your-profile
```

**Issue**: ADAM not found
```bash
# Solution: Verify bootstrap script ran successfully
cat /var/log/bootstrap-actions/1/stdout
```

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

MIT License - see LICENSE file for details

## 👤 Author

**Md Tariqul Islam**
- Email: tariqul@scired.com
- GitHub: [@mtariqi](https://github.com/mtariqi)
- LinkedIn: [mdtariqulscired](https://linkedin.com/in/mdtariqulscired)
- ORCID: [0009-0009-6545-8040](https://orcid.org/0009-0009-6545-8040)

## 🙏 Acknowledgments

- Apache ADAM community
- Apache Spark team
- Bioinformatics research community
- AWS for cloud infrastructure

## 📊 Citation

If you use this pipeline in your research, please cite:

```bibtex
@software{islam2024genomic,
  author = {Islam, Md Tariqul},
  title = {Genomic Variant Pipeline with ADAM and PySpark},
  year = {2024},
  url = {https://github.com/mtariqi/genomic-variant-pipeline}
}
```

---

⭐ **Star this repository if you find it helpful!**

**Keywords**: Genomics, Bioinformatics, ADAM, PySpark, Apache Spark, Variant Calling, AWS EMR, Big Data, Computational Biology
