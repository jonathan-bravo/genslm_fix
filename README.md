# README

[![Static Badge](https://img.shields.io/badge/conda-%E2%89%A54.12.0-%2344A833?logo=anaconda&logoColor=%2344A833)](https://anaconda.org/)


A fix for running [genslm](https://github.com/ramanathanlab/genslm).

## How to

Make sure that [conda](https://anaconda.org/) is installed.

```bash
git clone https://github.com/jonathan-bravo/genslm_fix.git

cd genslm_fix

make
```

The `make` command will create the conda environment at `genslm.yaml`, clone the
[genslm repository](https://github.com/ramanathanlab/genslm), correct both the
`setup.cfg` and the `neox_25,076,188,032.json` files and install genslm.

## Running genslm

First you must download one of the model files from the [Globus Endpoint](https://app.globus.org/file-manager?origin_id=25918ad0-2a4e-4f37-bcfc-8183b19c3150&origin_path=%2F).

Make sure the file is in the `genslm_fix` directory or that you point the code
to the correct path.

To test that genslm is working run the following code from the genslm repo:

```python
import torch
import numpy as np
from torch.utils.data import DataLoader
from genslm import GenSLM, SequenceDataset

# Load model
model = GenSLM("genslm_2.5B_patric", model_cache_dir=".")
model.eval()

# Select GPU device if it is available, else use CPU
device = "cuda" if torch.cuda.is_available() else "cpu"
model.to(device)

# Input data is a list of gene sequences
sequences = [
    "ATGAAAGTAACCGTTGTTGGAGCAGGTGCAGTTGGTGCAAGTTGCGCAGAATATATTGCA",
    "ATTAAAGATTTCGCATCTGAAGTTGTTTTGTTAGACATTAAAGAAGGTTATGCCGAAGGT",
]

dataset = SequenceDataset(sequences, model.seq_length, model.tokenizer)
dataloader = DataLoader(dataset)

# Compute averaged-embeddings for each input sequence
embeddings = []
with torch.no_grad():
    for batch in dataloader:
        outputs = model(
            batch["input_ids"].to(device),
            batch["attention_mask"].to(device),
            output_hidden_states=True,
        )
        # outputs.hidden_states shape: (layers, batch_size, sequence_length, hidden_size)
        # Use the embeddings of the last layer
        emb = outputs.hidden_states[-1].detach().cpu().numpy()
        # Compute average over sequence length
        emb = np.mean(emb, axis=1)
        embeddings.append(emb)

# Concatenate embeddings into an array of shape (num_sequences, hidden_size)
embeddings = np.concatenate(embeddings)
embeddings.shape
```
Make sure that the correct model is selected as the first argument of `GenSLM()`
and that the `model_cache_dir` is pointed to the path that has your downloaded
model files.

## Errors with conda

If you are getting an error during the make process because a conda package is
corrupt, make sure to clear the conda package cache.

```bash
conda clean --packages --tarballs -y
```

## Citations

If you are going to use one of the models, be sure to cite the original authors'
paper and their GitHub (if applicable).

```bibtex
@article{zvyagin2022genslms,
  title={GenSLMs: Genome-scale language models reveal SARS-CoV-2 evolutionary dynamics.},
  author={Zvyagin, Max T and Brace, Alexander and Hippe, Kyle and Deng, Yuntian and Zhang, Bin and Bohorquez, Cindy Orozco and Clyde, Austin and Kale, Bharat and Perez-Rivera, Danilo and Ma, Heng and others},
  journal={bioRxiv},
  year={2022},
  publisher={Cold Spring Harbor Laboratory}
}
```