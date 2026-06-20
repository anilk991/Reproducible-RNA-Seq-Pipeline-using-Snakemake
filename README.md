# Reproducible-RNA-Seq-Pipeline-using-Snakemake

## RNA-seq Alignment and Feature Counting Pipeline

This repository contains two Snakemake workflows designed to process Single-End (SE) and Paired-End (PE) RNA-seq data. The pipeline automates the journey from raw FASTQ files to gene-level read counts using HISAT2 and featureCounts.

---

### Pipeline Overview

Both workflows share a core structure but are tailored to handle the specific input requirements of PE and SE sequencing reads. The pipeline executes the following steps:

* **Quality Control (Optional):** Pre-processes raw FASTQ files using AfterQC to generate cleaned `.good.fq.gz` files. Note: This step is currently commented out in the scripts but is ready for activation.


* **Read Alignment:** Maps the cleaned reads to the human reference genome using HISAT2 and the `genome_tran` index.


* **Format Conversion:** Converts the initial SAM files into binary BAM format using Samtools.


* **Sorting:** Sorts the BAM files by coordinate using Samtools.


* **Indexing:** Generates `.bai` index files for the sorted BAMs using Samtools.


* **Quantification:** Counts mapped reads against the human GRCh38.112 GTF annotation using `featureCounts`.


* **Storage Optimization:** Automatically removes intermediate SAM files and unsorted BAM files to save disk space after they are no longer needed.



---

### Configuration and File Paths

Before running the pipeline, you must configure the directory variables at the top of the respective `Snakefile`. The default configuration uses the following paths:

| Variable | Paired-End (PE) Path | Single-End (SE) Path |
| :--- | :--- | :--- |
| **FASTQ_DIR** | `/media/SSD/Anil/ENA/GSE148434/PE` | `/media/SSD/Anil/ENA/GSE148434/SE` |
| **OUTPUT_DIR** | `/media/SSD/Anil/ENA/GSE148434/PE/good` | `/media/SSD/Anil/ENA/GSE148434/SE/good` |
| **COUNT_DIR** | `/media/SSD/Anil/ENA/GSE148434/PE/featureCounts` | `/media/SSD/Anil/ENA/GSE148434/SE/featureCounts` |
| **INDEX_FILE** | `/media/SSD/Anil/ENA/Index/Hisat2/genome_tran` | `/media/SSD/Anil/ENA/Index/Hisat2/genome_tran` |
| **GTF_FILE** | `/media/SSD/Anil/ENA/Index/Hisat2/Homo_sapiens.GRCh38.112.gtf` | `/media/SSD/Anil/ENA/Index/Hisat2/Homo_sapiens.GRCh38.112.gtf` |
| **THREADS** | `128` | `128` |

---

### Key Differences Between Workflows

While the underlying logic is identical, the commands adapt to the sequence read geometry:
| Step | Paired-End (PE) | Single-End (SE) |
| :--- | :--- | :--- |
| **Sample Parsing** | Uses a wildcard to detect paired files matching `{sample}_1.fastq.gz` and `{sample}_2.fastq.gz`. | Uses a wildcard to detect single files matching `{sample}.fastq.gz`. |
| **HISAT2** | Uses the `-1` and `-2` flags to input forward and reverse reads. | Uses the `-U` flag to input unpaired reads. |
| **featureCounts** | Includes the `-p` flag to instruct the tool to count fragments (read pairs) instead of individual reads. | Omits the `-p` flag. |

---

### Software Dependencies

To execute these pipelines, ensure the following bioinformatics tools are installed and available in your system's PATH:

* **Snakemake:** For workflow management.
* **AfterQC:** For read cleaning (Note: Requires Python 2).
* **HISAT2:** For splice-aware read alignment.
* **Samtools:** For manipulating SAM/BAM formats.
* **Subread package:** Specifically for the `featureCounts` utility.

---

### Usage Instructions

1. Clone this repository to your local machine.
2. Open the desired Snakemake file (PE or SE) and adjust the `FASTQ_DIR`, `OUTPUT_DIR`, `INDEX_FILE`, `COUNT_DIR`, and `GTF_FILE` variables to point to your local directories.
3. If you intend to use the AfterQC read cleaning step, uncomment the execution lines inside the `rule afterqc` block.
4. Execute the Snakemake pipeline via the command line by specifying the respective file and the number of cores to use:
* **For Single-End:** `snakemake -s snakefile_SE --cores <number_of_cores>`
* **For Paired-End:** `snakemake -s snakefile_PE --cores <number_of_cores>`


### Directory Structure: Paired-End (PE) Workflow

```text
/media/SSD/Anil/ENA/
├── Index/
│   └── Hisat2/
│       ├── genome_tran.*.ht2                      ← HISAT2 reference index files
│       └── Homo_sapiens.GRCh38.112.gtf            ← Reference annotation (GTF)
└── GSE148434/
    └── PE/
        ├── <sample>_1.fastq.gz                    ← Raw forward reads (Input)
        ├── <sample>_2.fastq.gz                    ← Raw reverse reads (Input)
        ├── good/                                  ← OUTPUT_DIR: AfterQC processed reads
        │   ├── <sample>_1.good.fq.gz              ← Cleaned forward reads
        │   └── <sample>_2.good.fq.gz              ← Cleaned reverse reads
        └── featureCounts/                         ← COUNT_DIR: Alignment and quantification
            └── <sample>/                          ← Per-sample output folder
                ├── <sample>_summary.txt           ← HISAT2 alignment summary log
                ├── <sample>_sorted.bam            ← Sorted BAM file
                ├── <sample>_sorted.bam.bai        ← Indexed BAM file
                └── <sample>.txt                   ← Final featureCounts read counts

```

*(Note: Intermediate `.sam` and unsorted `.bam` files are temporarily created in the `<sample>/` folder during execution but are automatically deleted by the pipeline to save space.)*

---

### Directory Structure: Single-End (SE) Workflow

```text
/media/SSD/Anil/ENA/
├── Index/
│   └── Hisat2/
│       ├── genome_tran.*.ht2                      ← HISAT2 reference index files
│       └── Homo_sapiens.GRCh38.112.gtf            ← Reference annotation (GTF)
└── GSE148434/
    └── SE/
        ├── <sample>.fastq.gz                      ← Raw single-end reads (Input)
        ├── good/                                  ← OUTPUT_DIR: AfterQC processed reads
        │   └── <sample>.good.fq.gz                ← Cleaned single-end reads
        └── featureCounts/                         ← COUNT_DIR: Alignment and quantification
            └── <sample>/                          ← Per-sample output folder
                ├── <sample>_summary.txt           ← HISAT2 alignment summary log
                ├── <sample>_sorted.bam            ← Sorted BAM file
                ├── <sample>_sorted.bam.bai        ← Indexed BAM file
                └── <sample>.txt                   ← Final featureCounts read counts

```


