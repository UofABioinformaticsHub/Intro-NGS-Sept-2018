* TOC
{:toc}

# Gene Expression Analysis

## Read Summarisation

Now that we have bam files with our reads aligned to the genome, the next step in a classic RNA-seq analysis would be to perform differential gene expression.
This is where we look for genes that have different expression levels between two (or more) treatment groups.
In our data, we have young (6mth) and old (24mth) samples.

Before we do this though, we need to find which reads are aligned to which gene and we will then count how many reads match each gene.
As we learned at the start of the day, gene descriptions are often contained in `gtf` files.
The full set of gene descriptions would normally be found on the [Ensembl database](ftp://ftp.ensembl.org/pub/release-95/gtf/danio_rerio/), however, given that we only have chromosome 2 today, we've placed a edited version on your VM already.
This file can be found as `~/data/Danio_rerio.GRCz11.95.chr2.gtf.gz` and now is a good time to *copy this into your directory* `~/genomes/Drerio/`.
You can use `cp` in your terminal, or use the GUI interface within RStudio for this.

### Feature Counts

There are numerous methods for counting reads which align to genes.
Common tools include HTSeq and kallisto (which is actually an aligner as well), or this can even be done inside R itself.
Today we'll use `featureCounts` from the `Subread` suite of tools as we've found it to be the fastest and most accurate tool for this task.
Here's a snapshot of the lines we use for this from one of our RNASeq pipelines.

```
## Feature Counts - obtaining all sorted bam files
SAMPLES=`find ${ALIGNDIR}/bam -name "*out.bam" | tr '\n' ' '`

## Running featureCounts on the sorted bam files
featureCounts -Q 10 \
  -s 2 \
  --fracOverlap 1 \
  -T ${CORES} \
  -a ${GTF} \
  -o ${ALIGNDIR}/counts/counts.out \ 
  ${SAMPLES}
  
## Storing the output in a single file
cut -f1,7- ${ALIGNDIR}/counts/counts.out | \
    sed 1d > ${ALIGNDIR}/counts/genes.out
```

Let's talk through these three steps:

#### 1. Find our samples
{:.no_toc}

Note that we've used a different strategy for finding our files this time.
Instead of using `$(ls ...)`, we've used `find`.
This is probably a superior approach and can withstand difficult file paths more easily.
Output from `find` will give you each result on a new line, so after this we've used `tr` to `tr`anslate line breaks `\n` to spaces.

#### 2. Count our alignments
{:.no_toc}

Here we've called `featureCounts` and passed several parameters to it.
The help page is quite extensive for this tool, so feel free to browse it in your terminal or on pages 37-42 of [the manual pdf](http://bioinf.wehi.edu.au/subread-package/SubreadUsersGuide.pdf).

| Parameter | Meaning |
|:---------:|:------- |
| `-Q 10` | The minimum mapping quality (MAPQ) score for a read to be counted. These scores can vary between aligners, but this value should capture uniquely aligning reads only. These values are generally documented very poorly & cause much confusion. |
| `-s 2`  | Specifies a reverse stranded library. Only reads which map to the opposite strand as the gene will be counted. If the library is forward stranded you would set this to `-s 1`. |
| `--fracOverlap 1` | The fraction of a read which must overlap a feature. Here every base must overlap an exon |
| `-T ${CORES}` | The number of cores we wish to use |
| `-a ${GTF}` | The gene description file |
| `-o ${ALIGNDIR}/counts/counts.out` | The output file |
| `${SAMPLES}` | The input files |

#### 3. Tidy up our output
{:.no_toc}

The output from `featureCounts` contains a few columns which we ignore, even though others may find them useful (exon coordinates, gene lengths etc)
In the final line, we're using `cut` to just return columns 1, then everything beyond 7.
`sed` is then used to delete the first line.
This will give us data in a nice easy format for importing into R.

**Try and write a script that runs `featureCounts`**.
Remember to declare all your variables, and **create any directories you need for the output**.
