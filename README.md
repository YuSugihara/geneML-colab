# geneML-colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YuSugihara/geneML-colab/blob/main/geneML_Colab.ipynb)

Run [**geneML**](https://github.com/hexagonbio/geneML) — deep-learning gene
prediction for fungal genomes — on a free Google Colab GPU.

Give it a genome FASTA and you get back a GFF3 annotation, CDS sequences and
protein sequences as a single zip download. No local install, no Google Drive,
no command line.

## Open the notebook

Click the badge above, or use this link:

```
https://colab.research.google.com/github/YuSugihara/geneML-colab/blob/main/geneML_Colab.ipynb
```

**Every click gives you a fresh notebook.** Colab loads the file straight from
this repository and never writes anything back to it, so whatever you ran last
time is gone and the cells come up unexecuted. If you want to keep your own
edits, use `File` → `Save a copy in Drive`; that copy is separate and the badge
link keeps opening the clean version from this repository.

A fresh *page* does not always mean a fresh *machine* — Colab may reattach the
runtime you were using earlier. If you want to start from a completely clean
machine, use `Runtime` → `Disconnect and delete runtime` first.

## Quick start

1. In Colab, go to `Runtime` → `Change runtime type` and select **T4 GPU**.
2. Run **Cell 1** (setup). This installs geneML and confirms that TensorFlow can
   actually see the GPU. Run it once per session.
3. Run **Cell 2** (run). It asks you to upload your genome, then decompresses it
   if needed, runs geneML, prints a summary, and downloads the results as a zip.

To annotate another genome in the same session, just run Cell 2 again.

Input can be `.fasta`, `.fa` or `.fna`, plain or gzipped. Gzipped input is
decompressed automatically and the original file is left untouched.

## Settings in Cell 2

| Setting | Default | What it does |
| --- | --- | --- |
| `FASTA_PATH` | empty | Leave empty to upload from your computer. Set it to reuse a file already in the session, e.g. `/content/input/genome.fna.gz`. |
| `OUTPUT_PREFIX` | `genome` | Base name for the output files. |
| `GENE_ID_PREFIX` | `geneML` | Prefix for gene IDs in the GFF3. |
| `MAX_TRANSCRIPTS` | `1` | Transcripts per gene. `1` gives the primary transcript only; `5` also reports alternative isoforms. |
| `MIN_GENE_SCORE` | `dynamic` | Score threshold. `dynamic` calibrates on the whole input and needs at least 100 kb. |
| `CONTIGS_FILTER` | empty | Comma-separated contig IDs, to run on a subset. |
| `CORES` | `2` | Worker processes. Free Colab gives 2 vCPUs. |
| `FAST_INFERENCE` | `False` | Faster execution path. Produces identical output. |
| `USE_CPU_ONLY` | `False` | Disable the GPU. Same results, much slower. |
| `DOWNLOAD_RESULTS` | `True` | Download the zip when the run finishes. |
| `EXTRA_ARGS` | empty | Any other geneML flag, passed straight through, e.g. `--max-intron-size 1000`. |

## Outputs

The zip contains `<prefix>.gff3` (annotation), `<prefix>.genes.fna` (CDS
sequences), `<prefix>.proteins.faa` (proteins) and `<prefix>.log` (the geneML
run log). The GFF3 has no UTRs; CDS features match exon features apart from
phase.

## Credits

geneML is developed by Hexagon Bio (Lawrence Hon and Lisa Vader) and released
under GPL-3.0 at [hexagonbio/geneML](https://github.com/hexagonbio/geneML).
This repository only provides a Colab wrapper; please cite geneML itself in any
work that uses it.
