# Key File Types
{:.no_toc}

Before we move on to sequencing technologies, let's have look at a few important file types & methods that you're likely to come across.

* TOC
{:toc}

## Genome Browsers

Genome browsers are applications that provide a way to view, explore and compare genomic information in a graphical environment.

Genome browsers enable researchers to visualize and browse entire genomes with annotated data including gene prediction and structure, proteins, expression, regulation, variation, comparative analysis, and so on. Annotated data is usually from multiple diverse sources. They differ from ordinary biological databases in that they display data in a graphical format, with genome coordinates on one axis and the location of annotations indicated by a space-filling graphic to show the occurrence of genes and other features[¹](https://en.wikipedia.org/w/index.php?title=Genome_browser&oldid=889818322).

### UCSC
{:.no_toc}

Later today we'll use a genome browser running on our local machines (IGV), but a good one to start with is the web-based UCSC browser.
Click [this link](http://genome.ucsc.edu/cgi-bin/hgTracks?db=hg38&lastVirtModeType=default&lastVirtModeExtraState=&virtModeType=default&virtMode=0&nonVirtPosition=&position=chr1%3A11102837%2D11267747&hgsid=690299847_w2EtEiD6JAZeB5jX6o9m02x2hGaI) and you should see a slightly intimidating screen full of information.

You'll be able to see:

1. Genes with their transcript structure at the top, along with their position on the genome, as defined by GENCODE
2. RefSeq-based gene models
3. OMIM alleles
4. Gene expression from multiple tissues
5. A whole lot of other information...

Once you've got a handle on what's there, locate the `hide all` button and click that, which will just give the genomic region with no track information.
We can turn on a huge variety of "*tracks*" which contain genomic informationthat we may care about.
Let's start by turning on the GENCODE transcripts again.

Under the **Genes and Gene Predictions** section, find the *GENCODE v24* drop-down menu and click the arrow next to the word 'hide'.
Change this to 'full' and hit one of the `refresh` button you can see scattered across the page.
Now the transcripts will appear again in a less cluttered display.
Under the hood, the browser has used this information saved as a `BED` file, which enables us to define genomic regions in a convenient *tab-delimited* format.
We'll look at these in more detail soon.

As well as showing the transcript structure, we can also show simple genomic features like a SNP, so let's look for the **Variation** region down the page a little, then set the *Common SNPs(150)* track to *full* as well.
To make these changes appear, hit a *refresh* button again and the browser will now have this track showing.
This is pretty crazy, so we can condense this using the *pack* option. 
Try this then the *dense* and *squish* options to see what difference they all make.

If you haven't already tried it, you can click on any of the genomic features and you'll be taken to a page containing all the key information about that feature.
You can also drag your mouse over regions to zoom in, and can zoom out using the buttons at the top of the page.
Type the name of your favourite gene into the search box and you'll be able to find your way to that.
If you can't think of one, just enter *IL2RA* and you'll be taken to a page **full** of choices.
As we're using GENCODE 24, look for that list about half way down and select one of the isoforms you can see.
This will take you back to the browser, but just showing the region for the selected transcript.

Now we've had a brief exploration of the browser, let's look at some file types which will enable us to upload custom features, and which are useful an numerous stages during analysis using NGS data.



## BED Files

### The basic format
{:.no_toc}

These are a common file type for uploading your own data to the UCSC browser if you'd like to add a custom track with your own genomic features, and can also be imported into the IGV browser which we'll explore later in the session.
This format is best used for genomic regions which all represent the same type of feature (e.g. genes, promoters, sequence motifs etc).
They're also able to be used as input for numerous analytic tools, so are very useful to know about.
A full description of the format is available at: https://genome.ucsc.edu/FAQ/FAQformat.html#format1

BED files are also very commonly used for interacting with a variety of NGS-related tools.
We can use these to just obtain a subset of alignments from a larger file, to restrict variant calling to specific regions, etc.

The basic structure is a tab-separated file, with a minimum of **three mandatory columns** giving the Chromosome (`chrom`), start (`chromStart`) and end (`chromEnd`) positions.
In this way we can simply define genomic regions of interest that we have found in our analysis, and can visualise them.
As with the vast majority of the file types we'll come across, each line needs to have the same number of fields, with the exception of any header lines.
Unlike other file types, header lines in bed files **do not** start with a comment character but can only begin with the words `browser` or `track`.
The header lines are important in the context of genome browser custom annotation tracks, but most external tools will not tolerate their presence.

Let's start by forming our own bed file.
First `cd ~` to change directory to home and then use `nano` (`nano gk.bed`) or another text editor to create a file that looks like this.

```
track name="FOXP3 sites"
chrX	30671901	30672803
chrX	30691567	30692445
```

These two regions were obtained as [enriched for FOXP3 binding within the gene *GK*](https://www.ncbi.nlm.nih.gov/pubmed/20554955).

1. Now you've saved the file (see the bash workshop text editor [notes](https://uofabioinformaticshub.github.io/BASH-Intro/notes/3_sed_awk_grep.html#command-line-interface-cli-text-editors-for-small-ish-files)), head to the UCSC browser at [https://genome.ucsc.edu/cgi-bin/hgGateway](https://genome.ucsc.edu/cgi-bin/hgGateway). Ensure you are using the `hg38` genome build.
2. Enter the gene GK in the `Position/Search Term` text box, then just click on any of the links returned by the search.
3. Find the button labelled `hide all` and click it.
4. Under `Genes and Gene Predictions`, find `GENCODE v24` and select `full` using the drop-down menu

This should just give you a generic view of the gene *GK*.

Now we have this view:

1. Select the `manage custom tracks` button directly below the browser.
2. Upload your file `gk.bed` and follow the link `chrX`

This will give a additional track on the browser which shows these two regions.

### Additional Columns (Advanced material)
{:.no_toc}


extensible format, but can just say that or move elsewhere see here!

In addition to the mandatory columns, there are 9 optional fields which are able to be added.
The order of these is fixed and these are:

- `name`: The name of the 'feature'
- `score`: An integer between 0 and 1000. This can be used to control the the darkness of the greyscale for each region.
- `strand`: Can only take the values '.', '+' or '-'
- `thickStart`: If you wish to have a feature with thick & thin sections, such as exons & introns, this sets these values
- `thickEnd`: Same as above...
- `itemRGB`: Set the colour of the feature in RGB format. This must be three integers between 0 and 255, separated by commas. These correspond directly to the amount of red, green or blue so `255,0,0` would correspond to red at maximum, with blue and green off. This also requires `itemRgb="On"` to be supplied in the header line.
- `blockCount`, `blockSizes` and `blockStarts`: These are used to define exons (or other sub-regions) within a larger region.

Let's add some colours to our two FOXP3 regions.

```
browser position chrX:30671000-30693000
track name="FOXP3 sites" itemRgb="On"
chrX	30671901	30672803  Site1 0 . 30671901	30672803 255,0,0
chrX	30691567	30692445  Site2 0 . 30691567	30692445 0,0,255
```

Once you've uploaded this, right-click the track on the browser and make sure that you have the track set to `full`.
Notice that we didn't bother with the final three columns.

## GFF/GTF Files

There can be a little confusion about GFF and GTF files and these share some similarities with BED files.
GFF (General Feature Format) files have version2 and version3 formats, which are slightly different.
Today, we'll just look at GTF (General Transfer Format) files, which are best considered as GFF2.2, as restrictions are placed on the type of entries that can be placed in some columns.

Whilst BED files are generally for showing all the locations of a single type of feature, multiple feature types can be specified within one of these files.
Again, like BED files, fields are tab-separated with no line provided which gives the column names.
These are fixed by design, and as such explicit column names are not required.

1. **seqname** - name of the chromosome or scaffold; chromosome names can be given with or without the 'chr' prefix. Important note: the seqname must be one used within Ensembl, i.e. a standard chromosome name or an Ensembl identifier such as a scaffold ID, without any additional content such as species or assembly. See the example GFF output below.
2. **source** - name of the program that generated this feature, or the data source (database or project name)
3. **feature** - feature type name, can only take the values "CDS", "start_codon", "stop_codon", "5UTR", "3UTR", "inter", "inter_CNS", "intron_CNS" and "exon" (CNS stands for Conserved Noncoding Sequence)
4. **start** - Start position of the feature, with sequence numbering starting at 1.
5. **end** - End position of the feature, with sequence numbering starting at 1.
6. **score** - A floating point value (i.e. decimal points are allowed)
7. **strand** - defined as + (forward) or - (reverse).
8. **frame** - One of '0', '1' or '2'. '0' indicates that the first base of the feature is the first base of a codon, '1' that the second base is the first base of a codon, and so on...
9. **attribute** - A *semicolon-separated* list of tag-value pairs, providing additional information about each feature. In the GTF format two mandatory features are required here, although they can be left blank:
    + **gene_id** *value*
    + **transcript_id** *value*

Notice that there's *no real way to represent our FOXP3 sites as a GTF file*!
This format is really designed for gene-centric features as seen in the 3rd column.
An example is given below.
Also note that header rows are not controlled, but must start with the comment character `#`

```
# Data taken from http://mblab.wustl.edu/GTF22.html
381 Twinscan  CDS          380   401   .   +   0  gene_id "001"; transcript_id "001.1";
381 Twinscan  CDS          501   650   .   +   2  gene_id "001"; transcript_id "001.1";
381 Twinscan  CDS          700   707   .   +   2  gene_id "001"; transcript_id "001.1";
381 Twinscan  start_codon  380   382   .   +   0  gene_id "001"; transcript_id "001.1";
381 Twinscan  stop_codon   708   710   .   +   0  gene_id "001"; transcript_id "001.1";
```

**Note**: People variously use GFF and GTF to talk about GFF version 2, and GFF to talk about GFF version 3. GFF2 is not compatible with GFF3, so make sure you have the correct file format if you are given a GFF file. There are conversion tools available to inter-covert them, they are rarely reliable.

## VCF Files

The most **hated** format is a VCF file, which stands for *Variant Call Format*, but is more accurately known as *Very Confusing Format*.
Again, the general structure is header rows (beginning with the double comment symbol `##`), followed by tab-separated columns with the actual data.
In this case, column names are provided directly about the data in a line starting with a single comment character (`#`).

Whilst a flexible format, it is heavily structured with abbreviations and symbols with important meaning, e.g. phased genotypes are separated by `|`, whilst unphased ones are separated by `/`.
The example is taken from the file specification at https://samtools.github.io/hts-specs/VCFv4.2.pdf, and we could spend an enormous amount of time unpacking this example.

Important things to note are:

- The first 8 columns are mandatory
- The `FORMAT` column defines the format of subsequent columns
- *Sample level* information follows the `FORMAT` column
- The header rows are called *Meta-information* rows, and describe the coding used in each field via a series of tags.

## FASTA Files

Most of us have seen these, and the basic format is very simple.
Information about a sequence is placed after a `>` symbol, and these can occur throughout the file, indicating the start of a new sequence.
Following these lines are simple sequence data to a width of either 70 or 80 characters.
Sequence data can be DNA, RNA or Amino Acid data

```
>HSGLTH1 Human theta 1-globin gene fragment
CCACTGCACTCACCGCACCCGGCCAATTTTTGTGTTTTTAGTAGAGACTAAATACCATATAGTGAACACCTAAGA
CGGGGGGCCTTGGATCCAGGGCGATTCAGAGGGCCCCGGTCGGAGCTGTCGGAGATTGAGCGCGCGCGGTCCCGG
GATCTCCGACGAGGCCCTGGACCCCCGGGCGGCGAAGCTGCGGCGCGGCGCCCCCTGGAGGCCGCGGGACCCCTG
GCCGGTCCGCGCAGGCGCAGCGGGGTCGCAGGGCGCGGCGGGTTCCAGCGCGGGGATGGCGCTGTCCGCGGAGGA
CCGGGCGCTGGTGCGCGCCCTGTGGAAGAA
```

This is the format genomes are provided in by all genomic repositories such as Ensembl, NCBI and the UCSC.
Each chromosome is specified by the header, with the entire sequence following.

## FASTQ Files

These are the extension of FASTA files which we usually obtain as output from our sequencing runs.
We'll spend some time exploring these later today.

## SAM Files

These are plain text **S**equence **A**lignment/**M**ap files, which we will also spend some time looking at later today.
The binary version of a SAM file is known as a BAM file, and is the plain text information converted to the more computer-friendly binary format.
This usually results in a size reduction of around 5-10 fold, and BAM files are able to be processed much more quickly by NGS tools.
We'll also have a good look at these during the course of the day.
