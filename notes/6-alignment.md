* TOC
{:toc}

# Sequence Alignment

Once we have cleaned our data of any contaminating sequences, and removed the bases which are more likely to contain errors, we can more confidently align our reads to a reference. Different experiments may have different reference sequences depending on the context. For example, if we have a sub-sample of the genome associated with restriction sites like RAD-Seq, we would probably align to a reference genome, or if we have RNA-Seq we might choose to align to the transcriptome instead of the whole genome. Alternatively, we might be interested in *de novo* genome assembly where we have no reference genome to compare our data to.

## How Aligning Works

Most fast aligners in widespread public use are based on a technique called the Burrows-Wheeler Transform, which is essentially a way of restructuring, or indexing, the genome to allow very rapid searching. This technique comes from computer science and is really beyond the scope of what most of us need to know. The essence of it is that we have a very fast searching method, and most aligners use a seed sequence within each read to begin the searching. These seeds are then expanded outwards to give the best mapping to a sequence. There are many different alignment tools available today and each one will have a particular strength. For example, `bowtie` is very good for mapping short reads, whilst `bowtie2` or `bwa` are more suited to reads longer than 50bp.

### What’s the difference

Some key differences between aligners is in the way they index the genome, and in the a way they are equipped to handle mismatches and indels (insertions and deletions). Choosing an aligner can be a difficult decision with the differences often being quite subtle. Sometimes there is a best choice, other times there really isn’t. Make sure you’ve researched relatively thoroughly before deciding which to use.

Here is a selection of commonly used aligners.

| aligner     | target     | lengths    |
| ----------- | ---------- | ---------- |
| [bwa](https://github.com/lh3/bwa) | DNA/RNA | short read|
| [minimap2](https://lh3.github.io/minimap2/) | DNA/RNA | short/long read |
| [STAR](https://github.com/alexdobin/STAR) | RNA | short read |
| [kallisto](https://pachterlab.github.io/kallisto/about) | RNA | short read |
| [HISAT2](https://ccb.jhu.edu/software/hisat2/manual.shtml) | RNA/DNA | short read |

## Aligning our WGS reads

### Downloading a Reference Genome

To align any reads, we first need to download the appropriate (_i.e._ latest) genome and then we can build the index to enable fast searching via the Burrows-Wheeler Transform. Like we’ve seen in the previous sections, our reads today come from the nematode or Roundworm (*Caenorhabditis elegans*). 

**Note**: If you want the full genome sequence you can use the command-line program `curl` to download the *C. elegans* genome sequence. If `curl` doesn't work for you, you can always re-download the genome (like you can do with all model genomes) by opening Firefox and head to [ftp://ftp.ensembl.org/pub/release-90/fasta/caenorhabditis_elegans/](ftp://ftp.ensembl.org/pub/release-90/fasta/caenorhabditis_elegans/). 

For today's tutorial, we've given you just the sequence of chrI.
It may have been accidentally saved as the file `WGS` so if you have a file called `WGS` and can't see this file call an instructor over.

```
# Have a look at the first few lines
cd ~/Refs/Celegans
head chrI.fa
```

Note that the first line describes the following sequence and begins with a > symbol. We can use this to search within the file using regular expressions and print all of these description lines.


## Building an Index

As mentioned above read aligners improve their performance by constructing an index of the genome reference so that initial seed locations can be rapidly found for alignment extension.
The process of constructing an index can take a significant amount of time, although only needs to be performed once for each genome reference being used and so amortises over the period it is being used.

We will provide an index for the subsequent steps.

## Aligning the reads

<!--TODO(kortschak) Replace this with STAR instructions -->
Because we only have a small subset of the actual sequencing run, we should be able to run this alignment in a reasonable period of time.
First we'll create a folder to output the alignments.

```
cd ~/WGS
mkdir -p 03_alignedData/bam
```

```
cd ~/WGS/02_trimmedData/fastq
bwa mem -t 2 ~/Refs/Celegans/Celegans_chrI SRR2003569_sub_1.fastq.gz SRR2003569_sub_2.fastq.gz | samtools view -bhS -F4 -> SRR2003569_chI.bam
mv SRR2003569_chI.bam ../../03_alignedData/bam
```

Let’s break down this main command a little. The first part of the command:

```
bwa mem -t 2 ~/Refs/Celegans/Celegans_chrI SRR2003569_sub_1.fastq.gz SRR2003569_sub_2.fastq.gz
```

will align our compressed sequenced reads to the Celegans_chrI `bwa` index that we made. Usually you can create a SAM file (see next section) to store all the alignment data. SAM files however are text files which can take up a significant amount of disk space, so its much more efficient to pipe it to the `samtools` command and create a compressed binary SAM file (called BAM). To do this, we run the program `samtools`:

```
samtools view -bhS - > SRR2003569_chI.bam
```

In this context, `samtools` view is the general command that allows the conversion of the SAM to BAM. There is another more compressed version of the SAM file, called CRAM, which you can also create using `samtools` view. However, we will not use that today.

**Note:** By using the `-t 2` parameter, we can take advantage of modern computers that allow multi-threading or parallelisation. This just means that the command can be broken up into 2 chunks and run in parallel, speeding up the process. Check your computer's system settings, but you should be able to use at least 2 or 4 threads to run this alignment!
If using phoenix or another HPC, this can really speed things up.

From there we moved our alignments into a more appropriate directory.
We could've written to this directory directly, and would usually do this in a full analysis, but the command was already getting rather lengthy.

To find out information on your resulting alignment you can `samtools`:

```
cd ~/WGS/03_alignedData/bam
samtools stats SRR2003569_chI.bam
```

This is basically the same as another command `samtools flagstat`, but it gives additional information.

### Questions

1. How many reads aligned to our genome?

2. How many reads aligned as a pair?

3. What information does `samtools stats` provide that `samtools flagstat` does not?

4. How many aligned as a "proper" pair? ..what the hell is a proper pair anyway??

# Viewing the alignments

A common tool used for viewing alignments is IGV browser.
Before we can view the alignments we need to sort and index the alignments.

```
samtools sort SRR2003569_chI.bam > SRR2003569_chI.sorted.bam
samtools index SRR2003569_chI.sorted.bam
```

Now we can open IGV by entering `igv` in the terminal.
This will open in a new window which may take a moment or two.

```
igv
```

Once you've opened IGV, go to the `Genomes` menu and select `Load genome from file`.
Navigate to where you have `chrI.fa` and load this file.
Although this isn't the full genome, it will have everything we've aligned.

Now go to the `File` menu and select `Load from File` and navigate to your alignments.
Unfortunately you won't see anything until you zoom in.
This is so IGV doesn't hold the entire set of alignments in memory which would slow your computer to a stand-still.
Keep zooming in until some alignments appear then have a look around.

*What does all of the information mean when you hover over an alignment?*

We'll come back and have a look again after we've finished calling variants.
