# Propionate / GVHD microbiome analysis

PICRUSt2-based functional prediction and 16S analysis for the propionate manuscript
(Turan et al.). Everything is driven by a single notebook,
[`scripts/01_kate_propionate_pw.Rmd`](scripts/01_kate_propionate_pw.Rmd), which
produces the panels in Figures 1 and 4.

## Requirements

**R 4.4.x is what this project was built against**, and it is worth getting right
before anything else — this is the most common way the project fails to run.

`renv.lock` pins R 4.4.1 together with a set of Bioconductor 3.19 packages (DESeq2,
Biostrings, Maaslin2) that are tied to that R version. `renv` chooses its package
library by R minor version, so it looks in `renv/library/macos/R-4.4/` under R 4.4 and
in `renv/library/macos/R-4.5/` under R 4.5. **If you start the project under R 4.5,
renv points at a different, empty library and every `library()` call fails with "there
is no package called 'tidyverse'"** even though the packages are sitting right there in
the 4.4 folder.

Check your version with `R --version`, then either:

- **Use R 4.4** (recommended). On macOS the installed copies live under
  `/Library/Frameworks/R.framework/Versions/`; point RStudio at 4.4 via
  *Tools → Global Options → General → R version*.
- **Or run `renv::restore()` under whichever R you have.** This rebuilds the whole
  library for that version. It works, but it takes a long time and the Bioconductor
  packages will resolve to a release matched to your R rather than the 3.19 versions
  recorded in the lockfile, so package versions will not be identical to the ones the
  published analysis used.

As a stopgap, an existing 4.4-built library does load correctly under R 4.5 if you put
it on the path explicitly — the notebook has been run end to end this way:

```r
.libPaths(c("renv/library/macos/R-4.4/aarch64-apple-darwin20", .libPaths()))
```

**RStudio** is convenient but not required. Knitting to HTML needs pandoc, which
RStudio bundles; from a bare `Rscript` you can run `knitr::knit()` instead, which
executes every chunk without needing pandoc.

## Setup

1. **Clone or download the repository.** On GitHub, click the green **Code** button
   and choose **Download ZIP** if you would rather not use git.

2. **Restore the R environment.** Open `propionate.Rproj`, which activates `renv`,
   and then run:

   ```r
   renv::restore()
   ```

   This reads `renv.lock` and installs the exact package versions the analysis was
   run with. It takes a while the first time. `renv::status()` will tell you whether
   anything is still out of sync.

3. **Download the large PICRUSt2 files.** Two stratified contribution tables are too
   large to version here (~40 MB each), so they are not in the repository:

   - `data/picrust2_out_steady/pathways_out/path_abun_contrib.tsv.gz`
   - `data/picrust2_out_transplant/pathways_out/path_abun_contrib.tsv.gz`

   Download them from
   [PICRUSt2 output data](https://drive.google.com/file/d/1IfQi9TMPXJ9QyKjmX4EOgzn8JLyFKWNz/view?usp=drive_link),
   extract, and place the two `pathways_out` folders inside `data/picrust2_out_steady/`
   and `data/picrust2_out_transplant/`. The rest of the notebook runs without them;
   only the two "stratified output" sections (which produce the genus-contribution
   panels, Figures 1J and 4G) need them.

4. **Run the analysis.** Knit `scripts/01_kate_propionate_pw.Rmd`, or open it and run
   the chunks from the top. Figures are written to `results/`.

## How the repository is laid out

| Path | What it is |
|---|---|
| `scripts/01_kate_propionate_pw.Rmd` | The entire analysis |
| `data/` | Raw inputs — do not write generated files here |
| `data/derived/` | Files the notebook generates as PICRUSt2 input (git-ignored) |
| `results/` | All generated figures (git-ignored, recreated by running the script) |
| `renv.lock` | Pinned package versions |

Both `results/` and `data/derived/` are ignored by git because everything in them is
reproduced by running the notebook. The folders are created automatically by the
setup chunk, so a fresh clone works without any manual `mkdir`.

## Notes on reproducibility

**Run the notebook top to bottom.** Several object names (`data_full`, `plot_data`,
`meta`, `tax_raw`, `top_pathways`) are reused between the steady-state section and the
transplant section. Running chunks out of order can therefore analyze the wrong
experiment without any error. The chunks feeding manuscript figures now assert which
dataset they are holding, but the safest approach is still a clean knit in a fresh
session.

**The PERMANOVA is seeded.** `adonis2()` derives its p-values by permutation, so the
seed is fixed (`set.seed(20250602)`) both in the setup chunk and immediately before the
test itself. Without it the p-values drift between runs: repeated unseeded runs of the
Figure 4C test gave Treatment p = 0.001–0.002 and Timepoint × Treatment p = 0.002–0.004.
The seeded values are Timepoint p = 0.001, Treatment p = 0.001, interaction p = 0.002,
with R² of 0.43, 0.10 and 0.09. Note that with the default 999 permutations the smallest
attainable p-value is 0.001, so no term in this test can support a claim of "p < 0.001";
raise `permutations =` if a finer resolution is needed.

**Clinical scores are read from their source.** The day 7 GVHD scores used in Figure
4H are parsed directly from `data/Exp85 Propionate GVHD scores.xlsx` rather than typed
into the script, with an assertion that the parsed values match the published ones. If
that assertion fires, the spreadsheet layout changed and the parsing needs revisiting.

**Working directory.** All paths are relative to `scripts/`, which is where knitr runs
an Rmd from. The setup chunk stops with an explanatory error if the working directory
is anything else.

**The PICRUSt2 input-prep chunks are `eval=FALSE`.** PICRUSt2 itself was run
separately on Linux (conda, `picrust2_pipeline.py` 2.0.3-b) and its outputs ship with
the repository, so nothing downstream depends on re-running them. They are kept to
document how the inputs were built, and they write to `data/derived/` rather than over
the raw files.

**One caveat on the input data.** An earlier version of the FASTA-cleaning chunk wrote
its output back over its own input file. As a result `data/transplant_sv.seqs.fna` in
this repository is the *cleaned* version, with the ` size=` annotations already
stripped, and the original is not recoverable from git history.
`data/steady_sv.seqs.fna` is untouched and still carries the raw Zymo headers. This
does not affect any result — PICRUSt2 was run before the overwrite, and its outputs
are what the notebook reads — but it explains why the two FASTA files look different.
