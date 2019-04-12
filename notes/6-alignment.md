* TOC
{:toc}

# Sequence Alignment

Once we have cleaned our data of any contaminating sequences, and removed the bases which are more likely to contain errors, we can more confidently align our reads to a reference. Different experiments may have different reference sequences depending on the context. For example, if we have a sub-sample of the genome associated with restriction sites like RAD-Seq, we would probably align to a reference genome, or if we have RNA-Seq we might choose to align to the transcriptome instead of the whole genome. Alternatively, we might be interested in *de novo* genome/transcriptome assembly where we have no reference genome to compare our data to.

## How Aligning Works

Most fast aligners in widespread public use are based on a technique called the Burrows-Wheeler Transform, which is essentially a way of restructuring, or *indexing*, the genome to allow very rapid searching. This technique comes from computer science and is really beyond the scope of what most of us need to know. The essence of it is that we have a very fast searching method, and most aligners use a seed sequence within each read to begin the searching. These seeds are then expanded outwards to give the best mapping to a sequence. There are many different alignment tools available today and each one will have a particular strength. For example, `bowtie` is very good for mapping very short reads, whilst `bowtie2` or `bwa` are more suited to reads longer than 50bp.

### What’s the difference

Some key differences between aligners is in the way they index the genome, and in the a way they are equipped to handle mismatches and indels (insertions and deletions).
Choosing an aligner can be a difficult decision with the differences often seeming quite subtle, but with significant impacts.
Sometimes there is a best choice, other times there really isn’t.
Make sure you’ve researched relatively thoroughly before deciding which to use.

Here is a selection of commonly used aligners.

| aligner     | query     | lengths    |
| ----------- | ---------- | ---------- |
| [bwa](https://github.com/lh3/bwa) | DNA/RNA | short read|
| [minimap2](https://lh3.github.io/minimap2/) | DNA/RNA | long read |
| [STAR](https://github.com/alexdobin/STAR) | RNA | short read |
| [kallisto](https://pachterlab.github.io/kallisto/about) | RNA | short read |
| [HISAT2](https://ccb.jhu.edu/software/hisat2/manual.shtml) | RNA/DNA | short read |

## Aligning our RNA Seq reads

### Downloading a Reference Genome

To align any reads, we first need to download the appropriate (_i.e._ latest) genome and then we can build the index to enable fast searching via the Burrows-Wheeler Transform. Like we’ve seen in the previous sections, our reads today come from zebrafish (*Danio rerio*). 

Let's download the reference, but first we'll need to create a directory to hold it.
As this may be useful in multiple experiments, it makes sense to place it in a directory outside our current project.

```
cd ~
mkdir -p genomes/Drerio
```

**Note**: If you want the reference genome sequence you can use the command-line program `curl` to download the *D. rerio* genome sequence from the ftp server at [Ensembl](ftp://ftp.ensembl.org/pub/release-95/fasta/danio_rerio/dna/). 
The file you would normally need is Danio_rerio.GRCz11.dna.primary_assembly.fa.gz, however, today we have only given you the reads which map to Chromosome 2.
(Restricting ourselves to just this chromosome will make the genome indexing step much faster, and will also allow us to align everything in the time we have.)
Copy the link for the file Danio_rerio.GRCz11.dna.chromosome.2.fa.gz from your web browser and use `curl` to download this file to your `genomes/Drerio` directory.
Please make sure you keep the filename the same as the source filename.
This way we'll all be able to use the same commands below.

```
# Have a look at the first few lines
cd ~/genomes/Drerio
zcat Danio_rerio.GRCz11.dna.chromosome.2.fa.gz | head
```

Note that the first line describes the following sequence and begins with a > symbol and the rest contains all the sequence information.
This should all look quite familiar really.
Unfortunately, when building an index, STAR (which we'll be using today) needs an extracted fasta file, so we'll have to extract this file now
```
gunzip Danio_rerio.GRCz11.dna.chromosome.2.fa.gz
```


## Building an Index

As mentioned above read aligners acheive their performance by constructing an index of the reference genome so that initial seed locations can be rapidly found for alignment extension.
The process of constructing an index can take a significant amount of time, although only needs to be performed once for each genome being used and so becomes less of a burden if you're reusing the same genome over a period of time.

We're not too concerned with the details of this step as many references can be obtained, or are provided with a pre-built index.
On phoenix, a very incomplete collection is the in the folder `/data/biorefs` and we are working to populate this more thoroughly.
Just copy & paste the following, which should complete within about 2 minutes, and gives us an index that STAR will be able to use when running alignments.

```
STAR \
    --runMode genomeGenerate \
    --runThreadN 2 \
    --genomeDir ~/genomes/Drerio \
    --genomeFastaFiles ~/genomes/Drerio/Danio_rerio.GRCz11.dna.chromosome.2.fa
```

## Aligning the reads

Because we only have a small subset of the actual sequencing run, we should be able to run this alignment in a reasonable period of time.
First we'll return to our project folder

```
cd ~/agingRnaSeq
```

As we did with trimming our files, here's the basic command for aligining one of our samples.

```
STAR \
    --runThreadN 2 \
    --genomeDir ~/genomes/Drerio \
    --readFilesIn 1_trimmedData/fastq/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz \
    --readFilesCommand gunzip -c \
    --outFileNamePrefix 2_alignedData/bam/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_ \
    --outSAMtype BAM SortedByCoordinate 
```

This should complete within 2 minutes as well, so while it's running make sure you understand all of the commands given.

**Why have we specified `--readFilesCommand gunzip -c`**<details>
  This tells STAR to read files in using this command, and emables us to leave our fastq files compressed, saving hard drive (i.e. storage) space.
  </details>
  
**What to do you think the line `--outSAMtype BAM SortedByCoordinate` is doing?**<details>
  This is making sure we return a BAM file instead of a SAM file, which will be sorted by genomic position instead of being sorted by the order the reads were aligned in.
  We'll discuss SAM/BAM files in a minute.
  </details>
  
Also note that the returned BAM file will have the prefix we have given, followed by the suffix `Aligned.sortedByCoord.out.bam`.
This suffix is just what STAR adds to it's generate alignments

## SAM and BAM files

Now we have run a single alignment, let's look in the directory `2_alignedData/bam`.

```
ll 2_alignedData/bam
```

Here we'll see that BAM file we mentioned earlier and a few log files which STAR always produces.
These can actually be quite handy, but we won't really look at them much today.
However, let's move them into our `log` folder and have a quick squiz.

```
mv 2_alignedData/bam/*out 2_alignedData/log/
mv 2_alignedData/bam/*tab 2_alignedData/log/
cat 2_alignedData/log/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_Log.final.out 
```

As you can see, this is a relatively detailed summary of our alignments for that file.
For now, let's return to our actual alignments, as contained in the BAM file.

### What is a BAM file?

BAM stands for Binary AlignMent and these are our alignments stored in binary.
There is another type of file called a SAM (Sequence AlignMent) file which is in plain text, but SAM files can become very large and waste our precious storage resources.
BAM files are the exact same files stored in binary format which enables them to be much smaller.
This also has the added benefit of being faster for computers to read & perform operations on, as only humas read plain text.
Computers don't.

To look at the contents of a BAM file, we'll need the tool `samtools` which is one of the most heavily utilised command-line tools in the world of bioinformatics.
Ths tool as a series of commands which we can see if we just type `samtools` into our terminal.
The one we'll need at this point is `samtools view`, which enables us to take a BAM file and send it to `stdout` as plin text so we can read it.

Use tab autocomplete to enter the following line:

```
samtools view 2_alignedData/bam/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_Aligned.sortedByCoord.out.bam | head -n2
```

This will return our first two alignments in a tab-delimited format.
Although the lines are many columns wide and will probably be wrapped around in your terminal, you should be able to immediately spot the sequence identifier lines which came from our original fastq files.
This is sometimes referred to as the query name (or `QNAME`).
The remainder of the fields are given below.

| Field | Name | Meaning |
| ---- | ----- | ------- |
| 1 | QNAME | Query template/pair NAME |
| 2 | FLAG | bitwise FLAG (discussed later) |
| 3 | RNAME | Reference sequence (i.e. chromosome) NAME |
| 4 | POS | 1-based leftmost POSition/coordinate of clipped sequence |
| 5 | MAPQ | MAPping Quality (Phred-scaled) |
| 6 | CIGAR | extended CIGAR string |
| 7 | MRNM | Mate Reference sequence NaMe (`=` if same as RNAME) |
| 8 | MPOS | 1-based Mate POSition |
| 9 | TLEN | inferred Template LENgth (insert size) |
| 10 | SEQ | query SEQuence on the same strand as the reference |
| 11 | QUAL | query QUALity (ASCII-33 gives the Phred base quality) |
| 12 | OPT | variable OPTional fields in the format TAG:VTYPE:VALUE |


The internal representation of BAM files does not (for the most part) include tabs, but the information stored there is identical.

Several of these fields contain useful information, so looking the the first few lines which we displayed above, you can see that these reads are mapped in pairs as consecutive entries in the QNAME field are often (but not always) identical.
Most of these fields are self-explanatory, but some require exploration in more detail.


#### SAM Flags (Field 2)

These are quite useful pieces of information, but can be difficult at first look.
Head to http://broadinstitute.github.io/picard/explain-flags.html to see a helpful description.
The simplest way to understand these is that it is a bitwise system so that each description heading down the page increases in a binary fashion.
The first has value 1, the second has value 2, the third has value 4 and so on until you reach the final value of 2048.
The integer value contained in this file is the unique sum of whichever attributes the mapping has.
For example, if the read is paired and mapped in a proper pair, but no other attributes are set, the flag field would contain the value 3.

#### Questions
{:.no_toc}


1. *What value could a flag take if the read was 1 - paired; 2 - mapped in a proper pair; 3 - it was the first in the pair, and 4 - the alignment was a supplementary alignment.*
2. *Some common values in the bam file are 16, 256 and 272. Look up the meanings of these values.*


Things can easily begin to confuse people once you start searching for specific flags, but if you remember that each attribute is like an individual flag that is either on or off (_i.e._ it is actually a binary bit with values 0 or 1).
If you searched for flags with the value 1, you wouldn't obtain the alignments with the exact value 1, rather you would obtain the alignments for which the first flag is set and these can take a range of values.

A summary of some of the flag can be obtained using the command `samtools flagstat`

```
samtools flagtsat 2_alignedData/bam/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_Aligned.sortedByCoord.out.bam 
```

#### Questions

1. How many reads aligned to our genome?

2. What information does `samtools stats` provide that `samtools flagstat` does not?

### CIGAR strings

These give useful information about the type of alignment that has been performed on the read.
In the first few reads we called up earlier, most had the value `..M` where `..` is some number.
These are the perfect Matches, where the sequence has aligned exactly.
In particular, RNA Seq alignments will feature CIGAR strings with large numbers followed by `N`.
This represents a skipped region, and in this context is very likely to be where part of a read aligns to one exon, whilst the other part of the read aligns to another exon, making this commonly indicative of a spliced alignment.

The other abbreviations in common use are I (insertion), D (deletion) and S (soft-clipping).
Soft-clipping is a strategy used by aligners to mask mismatches, so these are often analagous to substitutions.
Hard-clipping (H) is an alternative strategy, but the difference between the two is beyond the scope of today.

*What is the interpretation of the first `CIGAR` string in your set of alignments?*

## Aligning All Samples

OK. Now it's time to write a script for aligning all of our samples.
Let's use a similar strategy as before remembering that the key values we need to find for each sample are:

- The input file
- The output prefix

This is actually a bit simpler than for `AdapterRemoval`, but remember the strategy we used of building our script up one line at a time to check that we'd specified everything correctly, before actually running our tool.

*See if you can figure out how to move the log files as part of your script too!*

<!--
# Viewing the alignments

A common tool used for viewing alignments is IGV browser.
Before we can view the alignments we need to sort and index the alignments.

```
samtools sort SRR2003569_chI.bam > SRR2003569_chI.sorted.bam
samtools index SRR2003569_chI.sorted.bam
```

Now we can open IGV by entering `igv.sh` in the terminal.
This will open in a new window which may take a moment or two.

```
igv.sh
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
-->
