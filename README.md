# cerberus

Cerberus is a set of tools designed to characterize and enhance transcriptome annotations. Currently cerberus can do the following:
* represent transcription start sites (TSSs) and transcription end sites (TESs) as bed regions rather than single base pair ends
* integrate intron chains from multiple transcriptome annotations (GTFs) to create a transcriptome of the union of them all
* integrate TSSs and TESs from multiple GTFs as well as from outside BED sources to create end annotations from the union of them all
* number intron chains, TSSs, and TESs found by their priority in a reference GTF
* use the enhanced intron chain and end sets to annotate an existing GTF transcriptome and to modify the GTF and corresponding abundance matrices to reflect the new naming scheme / identities of the transcripts

## Workflow

### Calling TSS / TES regions from a transcriptome
Create and merge end regions from a transcriptome annotation (GTF) file.

```
Usage: cerberus gtf-to-bed [OPTIONS]

Options:
  --mode TEXT      Choose tss or tes  [required]
  --gtf TEXT       GTF file  [required]
  -o TEXT          Output file name  [required]
  --dist INTEGER   Distance (bp) to extend regions on either side
                   Default: 50
  --slack INTEGER  Distance allowable for merging regions
                   Default: 50
  --help           Show this message and exit.
```

Example calls:
```bash
cerberus gtf-to-bed \
  --mode tss \
  --gtf tests/files/Canx.gtf \
  -o test_output/Canx_tss.bed \
  --dist 50 \
  --slack 50

cerberus gtf-to-bed \
  --mode tes \
  --gtf tests/files/Canx.gtf \
  -o test_output/Canx_tes.bed \
  --dist 50 \
  --slack 50
```

<!-- Calls to generate test files:
```bash
cerberus gtf-to-bed \
  --mode tss \
  --gtf tests/files/Canx.gtf \
  -o tests/files/Canx_tss.bed \
  --dist 50 \
  --slack 50

cerberus gtf-to-bed \
  --mode tes \
  --gtf tests/files/Canx.gtf \
  -o tests/files/Canx_tes.bed \
  --dist 50 \
  --slack 50
``` -->

### Calling unique intron chains from a transcriptome
Create a tab-separated file detailing unique intron chains present in a
transcriptome annotation (GTF) file.

```
Usage: cerberus gtf-to-ics [OPTIONS]

Options:
  --gtf TEXT  GTF file  [required]
  -o TEXT     Output file name  [required]
  --help      Show this message and exit.
```

Example call:
```bash
cerberus gtf-to-ics \
  --gtf tests/files/Canx.gtf \
  -o test_output/Canx_ics.tsv
```

<!-- Calls to generate test files:
```bash
cerberus gtf-to-ics \
  --gtf tests/files/Canx.gtf \
  -o tests/files/Canx_ics.tsv

cerberus gtf-to-ics \
  --gtf tests/files/Canx_1.gtf \
  -o tests/files/Canx_1_ics.tsv

cerberus gtf-to-ics \
  --gtf tests/files/Canx_2.gtf \
  -o tests/files/Canx_2_ics.tsv
``` -->

### Aggregate end regions from multiple bed files
Create consensus end regions from multiple bed files. The intent is for some
of these files to come from `cerberus gtf-to-bed`.

```
Usage: cerberus agg-ends [OPTIONS]

Options:
  --mode TEXT   Choose tss or tes  [required]
  --input TEXT  Path to file w/ path to BED files on each line or comma-
                separated  list of file paths; ordered by priority  [required]
  -o TEXT       Output file name  [required]
  --help        Show this message and exit.
```

Example calls:
```bash
cerberus agg-ends \
  --mode tss \
  --input tests/files/Canx_tss_beds.txt \
  -o test_output/Canx_tss_agg.bed

cerberus agg-ends \
  --mode tes \
  --input tests/files/Canx_tes.bed \
  -o test_output/Canx_tes_agg.bed
```

<!-- Calls to generate test files
```bash
cerberus agg-ends \
  --mode tss \
  --input tests/files/Canx_tss_beds.txt \
  -o tests/files/Canx_tss_agg.bed

cerberus agg-ends \
  --mode tes \
  --input tests/files/Canx_tes.bed \
  -o tests/files/Canx_tes_agg.bed
``` -->

### Aggregate intron chains from multiple intron chain files
Create consensus intron chain annotations from multiple intron chain files
(output from `cerberus gtf-to-ics`).

```
Usage: cerberus agg-ics [OPTIONS]

Options:
  --input TEXT  Path to file w/ path to ic files on each line OR comma-
                separated list of files paths; ordered by priority  [required]
  -o TEXT       Output file name  [required]
  --help        Show this message and exit.
```

Example call:
```bash
cerberus agg-ics \
  --input tests/files/Canx_1_ics.tsv,tests/files/Canx_2_ics.tsv \
  -o test_output/Canx_ic_agg.tsv
```

<!-- Calls to generate test files
```bash
cerberus agg-ics \
  --input tests/files/Canx_ics.tsv \
  -o tests/files/Canx_ic_agg.tsv
``` -->

### Compute triplet IDs for a transcriptome
Using the regions from `cerberus agg-ends` and the intron chains from
`cerberus agg-ics`, determine which end regions and intron chain each transcript
in the input GTF uses and output the results to an h5 transcriptome representation.

```
Usage: cerberus assign-triplets [OPTIONS]

Options:
  --gtf TEXT      GTF of isoforms to assign triplets to  [required]
  --ic TEXT       Intron chain file  [required]
  --tss_bed TEXT  Bed file of TSS regions  [required]
  --tes_bed TEXT  Bed file of TES regions  [required]
  -o TEXT         Output file name  [required]
  --help          Show this message and exit.
```

Example call:
```bash
cerberus assign-triplets \
  --gtf tests/files/Canx.gtf \
  --ic tests/files/Canx_ic_agg.tsv \
  --tss_bed tests/files/Canx_tss_agg.bed \
  --tes_bed tests/files/Canx_tes_agg.bed \
  -o test_output/Canx_triplet.h5
```

<!-- Calls to generate test files:
```bash
cerberus assign-triplets \
  --gtf tests/files/Canx.gtf \
  --ic tests/files/Canx_ic_agg.tsv \
  --tss_bed tests/files/Canx_tss_agg.bed \
  --tes_bed tests/files/Canx_tes_agg.bed \
  -o tests/files/Canx_triplet.h5
``` -->

### Update transcript ids
Using the map generated in `cerberus assign-triplets`, update the transcript ids
and transcript names that are used a TALON abundance matrix and GTF with the new
triplet versions of the transcript ids / names

```
Usage: cerberus replace-ids [OPTIONS]

Options:
  --map TEXT    transcript ID map from assign_triplets  [required]
  --gtf TEXT    GTF of isoforms
  --ab TEXT     TALON abundance file
  --collapse    collapse transcripts with the same triplets
  --opref TEXT  Output file prefix to save updated gtf / ab  [required]
  --help        Show this message and exit.
```

Example call:
```bash
cerberus replace-ids \
  --h5 tests/files/Canx_triplet.h5 \
  --gtf tests/files/Canx.gtf \
  --ab tests/files/Canx_abundance.tsv \
  --collapse \
  --opref test_output/Canx_triplet_updated
```

<!-- Calls to generate test files:
```bash
cerberus replace-ids \
  --h5 tests/files/Canx_triplet.h5 \
  --gtf tests/files/Canx.gtf \
  --ab tests/files/Canx_abundance.tsv \
  --collapse \
  --opref tests/files/Canx_triplet_updated
``` -->

## Utilites

### h5 to tsvs
By default as output from `assign-triplets`, cerberus writes a .h5 file with
4 different tables in it corresponding to
* Unique intron chains
* Unique TSS regions in bed format
* Unique TES regions in bed format
* Mapping of transcripts to their corresponding TSS, intron chain, and TES

If you wish to save tsv versions of each of these files for easier viewing,
you can use this utility to convert it.

```
Usage: cerberus h5-to-tsv [OPTIONS]

Options:
  --h5 TEXT     h5 transcriptome file output from cerberus assign-triplets
                [required]
  --opref TEXT  output file prefix  [required]
  --help        Show this message and exit.
```

Example calls:
```bash
cerberus h5-to-tsv \
  --h5 tests/files/Canx_triplet.h5 \
  --opref test_output/Canx
```

<!-- Calls to generate test files:
```bash
cerberus h5-to-tsv \
  --h5 tests/files/Canx_triplet.h5 \
  --opref tests/files/Canx_triplet
``` -->
