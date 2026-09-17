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
| `FAST_INFERENCE` | `False` | Faster execution path with identical output. See below. |
| `USE_CPU_ONLY` | `False` | Disable the GPU. Same results, much slower. |
| `DOWNLOAD_RESULTS` | `True` | Download the zip when the run finishes. |
| `EXTRA_ARGS` | empty | Any other geneML flag, passed straight through, e.g. `--max-intron-size 1000`. |

## Outputs

The zip contains `<prefix>.gff3` (annotation), `<prefix>.genes.fna` (CDS
sequences), `<prefix>.proteins.faa` (proteins) and `<prefix>.log` (the geneML
run log). The GFF3 has no UTRs; CDS features match exon features apart from
phase.

## Transcript variants

With `MAX_TRANSCRIPTS = 1` every gene gets exactly one mRNA, marked
`TranscriptVariant=PRIMARY`, so no isoforms are reported. Set it to `5` to keep
alternative transcripts; geneML labels each extra mRNA with the splicing event
that distinguishes it from the primary one (`INTRON_RETENTION`, `EXON_SKIPPING`,
`ALT_FIRST_EXON`, `ALT_LAST_EXON`, `ALT_5_SPLICE_SITE`, `ALT_3_SPLICE_SITE` or
`COMPLEX`). The gene count stays the same either way — only the number of mRNA
records changes. geneML caps the selection at 5, so larger values change nothing.

## Accuracy

This notebook never trades accuracy for speed. In particular it does **not**
enable float16 / mixed precision, which would be faster on a T4 but can shift
borderline scores.

`FAST_INFERENCE` changes only *how* the model is executed, not what it computes.
The same float32 weights run as a traced TensorFlow graph instead of eager
op-by-op dispatch, and sequences are scored in 200 kb chunks instead of 100 kb.
The chunking is exact, because each chunk is padded with 400 bp of context on
both sides, so every base keeps its full context regardless of chunk size. This
was verified by running both settings on the same input: the GFF3 and protein
FASTA files came out byte-identical. It is off by default so that a first run
reproduces earlier results exactly. To check on your own genome, run it once
each way and compare with `md5sum`.

## Performance on a free GPU

- **Confirm the GPU is really being used.** This is by far the biggest factor.
  If TensorFlow falls back to the CPU, geneML still runs but is roughly an order
  of magnitude slower. Cell 1 checks this explicitly and warns you.
- `TF_FORCE_GPU_ALLOW_GROWTH=true` is set for the run, so the worker processes
  share the single GPU instead of the first one reserving nearly all of its
  memory. This has no effect on the numbers produced.
- `CORES = 2` matches the 2 vCPUs of free Colab. It also pipelines the work:
  while one process is on the GPU, the other runs the CPU-bound gene-calling
  step.
- Cell 2 prints the wall-clock time and bp/s, so you can compare configurations
  on your own data. For quick timing experiments use `CONTIGS_FILTER`; note that
  `dynamic` scoring calibrates on the whole input, so a contig subset is for
  timing only, not for a final annotation.
- Free Colab disconnects idle sessions and caps total runtime, so keep the tab
  open for large genomes.

## Compatibility fix

geneML 1.1.0 fails with
`AttributeError: 'float' object has no attribute 'lower'` whenever
`--min-gene-score` is given a number instead of `dynamic`
(`args.py` parses the value to a float, then `params.py` calls `.lower()` on
it). Cell 2 runs geneML through a small launcher that passes the value through
as a string, so the numeric thresholds work. With `dynamic` the launcher changes
nothing, and any failure falls back to geneML's own behaviour.

## Repository layout

```
geneML-colab/
├── README.md
└── geneML_Colab.ipynb
```

Commit the notebook **without cell outputs** so it always opens clean and the
diffs stay readable:

```bash
pip install nbstripout
nbstripout --install        # run once inside the repository
```

## Credits

geneML is developed by Hexagon Bio (Lawrence Hon and Lisa Vader) and released
under GPL-3.0 at [hexagonbio/geneML](https://github.com/hexagonbio/geneML).
This repository only provides a Colab wrapper; please cite geneML itself in any
work that uses it.
