* TOC
{:toc}

# Key File Types

Before we move on to sequencing technologies, let's have look at a few important file types that you're likely to come across.

- bed
- gtf / gff
- vcf
- fasta
- fastq
- sam
- bam

## BED Files

### The basic format
{:.no_toc}

These are a common file type for uploading your own data to the UCSC browser if you'd like to add a custom track with your own genomic features.
They're also able to be used as input for numerous analytic tools, so are very useful to know about.
A full description of the format is available at: https://genome.ucsc.edu/FAQ/FAQformat.html#format1

The basic structure is a tab-separated file, with a minimum of three columns giving the Chromosome (`chrom`), start (`chromStart`) and end (`chromEnd`) positions.
In this way we can simply define genomic regions of interest that we have found in our analysis, and can visualise them.
As with the vast majority of the file types we'll come across, each line needs to have the same number of fields, with the exception of any header lines.
Unlike other file types, header lines in bed files **do not** start with a comment character but can only begin with the words `browser` or `track`.

Let's start by forming our own bed file.
The following two regions were obtained as enriched for FOXP3 binding within the gene *GK*[1](https://www.ncbi.nlm.nih.gov/pubmed/20554955).
Ensuring the data is tab-delimited, save the following as `gk.bed`.
You can use the text editor `gedit`, which we have installed on your VMs to do this.

```
track name="FOXP3 sites"
chrX	30671901	30672803
chrX	30691567	30692445
```

1. Head to the UCSC browser at https://genome.ucsc.edu/cgi-bin/hgGateway. Ensure you are using the `hg38` genome build.
2. Enter the gene GK in the `Position/Search Term` text box, then just click on any of the links returned by the search.
3. Find the button labelled `hide all` and click it.
4. Under `Genes and Gene Predictions`, find `GENCODE v24` and select `full` using the drop-down menu

This should just give you a generic view of the gene *GK*.

Now we have this view:

1. Select the `manage custom tracks` button directly below the browser.
2. Upload your file `gk.bed` and follow the link `chrX`

This will give a additional track on the browser which shows these two regions.

### Additional Columns
{:.no_toc}

In addition to the mandatory columns, there are 9 optional fields which are able to be added.
The order of these is fixed and these are:

- `name`: The name of the 'feature'
- `score`: An integer between 0 and 1000. This can be used to control the the darkness of the greyscale for each region.
- `strand`: Can only take the values '.', '+' or '-'
- `thickStart`: If you wish to have a feature with thick & thin sections, this sets these values
- `thickEnd`: Same as above...
- `itemRGB`: Set the colour of the feature in RGB format. This must be three integers between 0 and 255, separated by commas. Requires `itemRgb="On"` to be supplied in the header line.
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

## GTF Files
