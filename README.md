
# Gene Identification and Classification with DNABERT

This project fine-tunes a DNABERT model for two tasks on bacterial genomes:
1. **Gene Presence Detection** – Token-level binary classification (gene vs. non-gene).
2. **Gene Type Classification** – Token-level multi-label classification (e.g., CDS, rRNA, tRNA, etc.).

The project uses bacterial reference genomes from the [NCBI Genome Database](https://www.ncbi.nlm.nih.gov/datasets/genome/?taxon=2&reference_only=true) and performs token-level inference across millions of base pairs using a windowed k-mer embedding approach. The final outputted models can be used further for the same classification tasks on raw bacterial sequences; however, the models trained in the example code were trained on limited data and computational resources and are therefore only meant to serve as a proof-of-concept for what can be achieved with higher-level DNABERT fine-tuning. Training on larger datasets over more epochs, especially with a greater distribution of gene types (more even distribution of CDS to non-CDS genes) will likely yield better results and can serve more efficiently as a solution to bacterial gene detection and classification problems.

## Features

- Uses `zhihan1996/DNA_bert_6` (DNABERT) as the base transformer.
- Extracts and chunks raw genome sequences using overlapping sliding windows.
- Labels each k-mer based on gene annotations parsed from GenBank records taken from RefSeq-only reference GBFF files.
- Implements separate pipelines for:
  - Binary classification (gene presence)
  - Multi-label classification (gene type)
- Fine-tunes DNABERT using HuggingFace's `Trainer` and `BertForTokenClassification`.
- Evaluates models using precision, recall, F1-score (micro and macro), and confusion matrices.

## Directory Structure

```
data/downloads/               # Contains data files downloaded either manually or from NCBI or from the DirectDownload section of the project code
docs/                         # Contains project final report
Python/                       # Contains main project notebook and fine-tuned model parameters
├── project.ipynb             # Main notebook for training and evaluation
├── GeneDetectionModel/       # Checkpoints for gene presence model (Not included until run in code)
├── GeneClassificationModel/  # Checkpoints for gene type classification model (Not included until run in code)
.env                          # Contains NCBI Entrez API keys (optional)
LICENSE                       # Standard Apache 2.0 license
README.md                     # This file
```

## Requirements

- Python >= 3.8 (tested on 3.13.2)
- torch
- numpy
- transformers
- datasets
- scikit-learn
- intervaltree
- dotenv
- matplotlib
- biopython (Bio)

Install dependencies:
```bash
pip install -r requirements.txt
```

## Usage
### Setup .env
- Create a file named `.env` in root project directory (ex.: `DNABERT-Fine-Tuning-for-Bacterial-Gene-Identification-and-Classification/`) and add the following:
```
NCBI_EMAIL="example@email.com"
NCBI_API_KEY="YourNCBIEntrezAPIKey"
```

If you do not already have an Entrez API Key:
Register for an NCBI account at https://www.ncbi.nlm.nih.gov/account/ and create an Entrez API key under "API Key Management" in your account settings.
This will increase your rate limits and improve reliability for downloading larger sets of genomic data.

### 0. Initial parameters after imports
- useEntrez: True to use Entrez API to search NCBI Assembly database and download `maxResults` number of `.gbff` files; False if you want to skip those sections and either download data on your own or use eSearch implementation (not fully functional) 
- maxResults: Number of records to return from Entrez API eSearch
- directDownload: True if using Entrez eSearch or if files already present in `outDirectory`; false if using remote access via Entrez (again, not fully functional)
outDirectory: Directory to store downloaded GBFF files
testbasic: True if testing basic DNABERT on data for comparison (spoiler alert: it sucks)

### 1. Preprocess and Label Genome Data
- Optional: Use Entrez API to search Assembly database and extract `.gbff` files from `.gz` files
- Parse  `.gbff` files and extract genomes and other features
- Chunk sequences into overlapping sliding windows of 512 6-mers
- Use interval trees to assign binary and multi-class labels to each token

### 2. Train the Models
- (Optional) Base DNABERT with no modifications for comparison
- For gene detection:
  - Model: `BertForTokenClassification` with 2 output labels
- For gene type classification:
  - Model: `BertForTokenClassification` subclassed to use `BCEWithLogitsLoss`

### 3. Evaluate the Models
- Predictions are evaluated using:
  - `accuracy_score`
  - `precision_score`, `recall_score`, `f1_score`
  - `classification_report`
  - `multilabel_confusion_matrix`

### 4. Load from Checkpoints
To evaluate a previously trained model:
```python
from transformers import BertForTokenClassification
model = BertForTokenClassification.from_pretrained("GeneDetectionModel/")
```
or
```python
from transformers import BertForTokenClassification
model = BertForTokenClassification.from_pretrained("GeneClassificationModel/")
```

## Citation

If you use this project, please cite:
> Ji, Y., Zhou, Z., Liu, H., & Davuluri, R. V. (2021). DNABERT: pre-trained Bidirectional Encoder Representations from Transformers model for DNA-language in genome. *Bioinformatics, 37*(15), 2112–2120. https://doi.org/10.1093/bioinformatics/btab083

## Author

Justin Morera
