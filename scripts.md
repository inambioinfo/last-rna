# Scripts to analyze LAST RNA-genome alignments

These scripts compare RNA-genome alignments to gene annotations.  They
indicate: expression levels, novel exons, and gene fusions.

The scripts require [seg-suite](https://github.com/mcfrith/seg-suite)
to be installed.

They also require a file of gene annotations in UCSC format.  For
example, you can use `knownGene.txt` or `refGene.txt` from here:
<http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/>.  Be sure
to use the *same* genome version for alignment and annotation!

The scripts assume that the RNA sequences are of unknown/mixed
strands.  So they compare the RNAs to gene annotations on both
strands.  (Actually, strandedness is often indicated by splice signals
such as `gt-ag`: currently the scripts ignore this information.)

## Nicer gene names

`knownGene.txt` has a unique "name" (such as `uc003nrl.4`) for each
isoform.  There are nicer gene names (such as `TUBB`) in `kgXref.txt`
(also at <http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/>).
The following command prepends the gene name (with a comma separator)
to each isoform name:

    kgName.sh knownGene.txt kgXref.txt > genes.txt

Often, one gene name has multiple isoforms.

## Gene overlaps & expression

You can find gene overlaps like this:

    gene-overlap.sh genes.txt alignments.maf > overlap.txt

This gets the isoform with most overlapping bases for each RNA.  (If
there is more than one such isoform, it gets the one with
ascii-betically earliest gene/isoform name.)  You can then get rough
expression levels by counting the number of RNAs for each gene.

Counts per isoform:

    tail -n+2 overlap.txt | cut -f3 | uniq -c

Counts per gene:

    tail -n+2 overlap.txt | cut -f3 | cut -d, -f1 | uniq -c

## Novel exons

You can find novel exons of known genes like this:

    new-exons.sh genes.txt alignments.maf > exons.bed

The output is in BED format, which can be viewed in genome browsers.
The 4th column contains: the gene name, then the RNA identifier,
separated by a colon.

## Gene fusions

You can find gene fusions like this:

    gene-fusions.sh genes.txt alignments.maf > fusions.txt

The output has 7 columns.

* Column 1: RNA identifier.
* Column 2: RNA length (number of genome bases covered by its exons).
* Column 3: Splicing pattern (see below).
* Column 4: The gene/isoform with most overlapping bases.
* Column 5: The number of overlapping bases.
* Column 6: The gene/isoform with most overlap to the remaining bases.
* Column 7: The number of remaining bases that overlap.

It includes "fusions" between two isoforms of one gene, which may be
less interesting.  You can remove them like this:

    awk -F'\t|,' '$4 != $7' fusions.txt > out.txt

The column 3 splicing patterns look like this:

    S:4t3
    S:6b19
    S:7
    U:4t1

`4t3` means there are 4 typical exons, then a "trans" splice
(e.g. between chromosomes), then 3 typical exons.

`6b19` means there are 6 typical exons, then a "big" splice (more than
1 million bases), then 19 typical exons.

`U:` means that there is an unspliced part (i.e. a `1`).  These are
less reliable, because they may involve misalignment to a processed
pseudogene.  You can remove them like this:

    grep -v U: fusions.txt > out.txt
