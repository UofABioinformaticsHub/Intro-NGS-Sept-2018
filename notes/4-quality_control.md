* TOC
{:toc}

# Organising Your Data

Before we move into the actual analysis section of today, let's talk briefly about how to structure your analysis.
During NGS analysis, we will ended up creating multiple files and running multiple scripts, possibly in both `R` and `bash`.
Any experienced bioinformatician will have their own approach to organising everything, as we all know how easily your directories can get messay and out of control.

An example of how some in the Bioinformatics Hub structure their experiments is located at https://github.com/UofABioinformaticsHub/ngsSkeleton.
Let's copy this to our VMs and have a look at what we have.

<!-- Dan, can you please change this so it uses curl? I can't figure it out... -->
```
cd ~
wget https://github.com/UofABioinformaticsHub/ngsSkeleton/archive/master.zip
unzip master.zip
mv ngsSkeleton-master agingRnaSeq
rm master.zip
ls
```

**Make sure you understand what each line of the above has done. Ask a tutor if you need help.**

This will have downloaded a generic directory tree and placed all directories in a parent folder called `agingRnaSeq`.
Now let's look at what we have in our new directory.

```
cd agingRnaSeq
ls
```

This will list the first level directories in our main location for our experiment: `0_rawData  1_trimmedData  2_alignedData  R  README.md  bash  slurm`.
The first three are places we can store our raw, trimmed and aligned data.
We also have folders for `bash` and `R` scripts. **Why might we do this?**.
The `slurm` folder isn't a cheap Futurama reference, but is where can store the output from the queuing system called `slurm`, which is used by `phoenix`.
We won't really use this folder today though.

Look inside the `0_rawData` directory:

```
ls 0_rawData
```

Here we have the folders `fastq` and `FastQC` where we can store our actual `fastq` files, and our QC reports (which we're about to generate) can be placed in the `FastQC` folder.
Notice that we've used lower and upper-case `f/F` values to being each sub-directory name, because that's actually correct by file and tool names that we'll be using, and it really helps with easy tab-autocompletion.

You'll see similar structures inside the `1_trimmedData` and `2_alignedData` directories.
Have a look if you'd like, and notice the extra directories such as `log` and `bam`.
We'll get to all of these over the next hour or two.

Before beginning our QC, let's copy our files into the folder we've just created.

```
cd ~/agingRnaSeq
cp ~/data/aging_study/* ./0_rawData/fastq/
```

Notice our convenient use of the wildcard `*` which we came across yesterday which allowed us to copy all the files pretty easily.
This is one of the reasons that bioinformaticians love to work in bash or a command-line environment.
Now we've organised ourselves, we're ready to QC our data.

# Quality Control

## Using FastQC

A common tool for checking the quality of a FASTQ file is the program FastQC.
As with all programs on the command line, we need to see how it works before we use it.
The following command will open the help file in the less pager which we used earlier.
To navigate through the file, use the `<spacebar>` to move forward a page, `<b>` to move back a page & `<q>` to exit the manual.

```
fastqc -h | less
```

FastQC will create an html report for each file you provide, which can then be opened from any web browser such as firefox.
As seen in the help page, FastQC can be run from the command line or from a graphic user interface (GUI).
Using a GUI is generally intuitive so today we will look at the command line usage, as that will give you more flexibility & options going forward.
Some important options for the command can be seen in the manual.
As you will see in the manual, setting the `-h` option as above will call the help page.
Look up the following options to find what they mean.

| Option | Usage |
|:------ |:------|
| `-o`     |       |
| `-t`     |       |

The VMs we're all using have two cores so we can set the parameter `-t 2`.
We can also write to the output folder that we've already created above.
Let's run FastQC on all of our files and see what we get.
(Make sure you're in the `agingRnaSeq` folder first.)

```
fastqc -t 2 -o 0_rawData/FastQC/ 0_rawData/fastq/*
```

This will have created a FastQC report for every file as an html file which we can view using any web browser.

```
ls 0_rawData/FastQC/
```

However, we're on a VM today and we've discreetly avoided giving you a GUI/Desktop interface (they're clunky).
This leaves two options, 1) copying them to your local machine (i.e. laptop) using `scp`, or 2) using RStudio to have a look.
Let's take this second option, which may prove to be convenient as we go along too.

## Opening RStudio

As we did on Monday, open your regular internet browsser on your laptop and head to your IP address, but make sure you add the port `:8787` to the end of your IP.
This will open RStudio in your browser, so login using your username (`trainee`) and the password you changed it to on Monday.

As we learned Monday, R Projects can be very useful for helping keep an analysis organised, so let's create a new one for our RNA-Seq dataset.
Once you're in RStudio, go to `File > New Project > Existing Directory` then browse to `~/agingRnaSeq` and select `Create Project`.
This will open you up in the project root directory and you'll see all the folders we placed there earlier.
Using the `Files` pane, navigate to `0_rawData/FastQC`.

Here you'll see all those files we created by running the `FastQC` tool, so select one of the `html` files, and when asked `Open in Editor` or `View in Web Browser`, choose the `Web Browser` option.
This will open the FastQC report in a new tab in your browser.
If you have a popup blocker, please allow this tab to open and you'll be able to see what the FastQC report looks like for your given file.


## Inspecting a FastQC Report

The left hand menu contains a series of click-able links to navigate through the each module contained in the report, with a quick guideline about each module given as a tick, cross or exclamation mark.
Some of these are not particularly informative, and the modules we can reasonably ignore in the vast majority of cases are:

1. Per tile sequence quality
2. Per sequence quality scores
3. Per base N content
4. Kmer Content (if it's even included)

#### Questions
{:.no_toc}

1. *How many sequences are there in both files?*
2. *How long are the sequences in these files?*

## Interpreting the FastQC Report

As we work through the QC reports we will develop a series of criteria for cleaning up our files.
There is usually no perfect solution, we just have to make the best decisions we can based on the information we have.
Some sections will prove more informative than others, and some will only be helpful if we are drilling deeply into our data.
Firstly we’ll just look at a selection of the plots.
We’ll investigate some of the others with some ‘bad’ data later.

### Per Base Sequence Quality
{:.no_toc}

Click on the `Per base sequence quality` hyper-link on the left of the page & you will see a boxplot of the QC score distributions for every position in the read.
These are the PHRED scores we discussed earlier, and this plot is usually the first one that bioinformaticians will look at for making informed decisions about the overall quality of the data and settings for later stages of the analysis.

*What do you notice about the QC scores as you progress through the read?*

We will deal with trimming the reads in a later section, but start to think about what you should do to the reads to ensure the highest quality in your final alignment & analysis.

As this is RNA-Seq data, we'll mainly be quantifying the number of reads which align to a gene. 
**Will the actual sequence content be vital for us?**. 
Now consider a whole-genome-sequencing (WGS) experiment where we are wanting to identify SNPs or other genomic variants.
Would the actual sequence content be vital for us now?

**Per Tile Sequence Quality**<details>
This section just gives a quick visualisation about any physical effects on sequence quality due to the tile within the each flowcell or lane.
Generally, this would only be of note if drilling deeply to remove data from tiles with notable problems.
Most of the time we don’t factor in spatial effects, unless alternative approaches fail to address the issues we are dealing with.
</details>

**Per Sequence Quality Scores**<details>
This is just the distribution of the average quality scores for each read, obtained by averaging all the scores at each base within a read..
There’s not much of note for us to see here.
</details>

**Per Base Sequence Content**<details>
This will often show artefacts from barcodes or adapters early in the reads, before stabilising to show a relatively even distribution of the bases.
  Here FastQC may have flagged this as a 'fail' and you will see considerable variability in the first few bases.
  This is actually very normal for RNA seq and is a consequence of non-random fragmentation and non-random adapter ligation.
  It's also relatively common to see a drift towards G towards the end of a read. This can be a bit more troubling (ask a tutor to explain) but is usually remedied as we trim our reads in the next step.
</details>

**Sequence Length Distribution**<details>
This shows the distributions of read lengths in our data. Here we have reads that are pretty much all the same length, however if the length of your reads is vital (_e.g._ smallRNA data), then this can also be a very informative plot.
</details>

**Sequence Duplication Levels** This plot shows about what you’d expect from a typical NGS experiment.
There are a few duplicated sequences (rRNA, highly expressed genes etc.) and lots of unique sequences representing the diverse transcriptome.
This is only calculated on a small sample of the library for computational efficiency and is just to give a rough guide if anything unusual stands out.
Things to watch for here are peaks on the far right which would indicate massive overrepresentation of a few sequences above the rest of the source material.

**Overrepresented Sequences** Here we can see any sequence which are more abundant than would be expected. Sometimes you'll see sequences here that match the adapters used, or you may see highly expressed genes here.

**Adapter Content** This can give a good guide as to our true fragment lengths. If we have read lengths which are longer than our original DNA/RNA fragments (_i.e._ inserts) then the sequencing will run into the adapters.
If you have used custom adapters, you may also need to supply them to `FastQC` as this only searches for common adapter sequences.

## Some More Example Reports

Let’s head to another sample plot at the [FastQC homepage](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/bad_sequence_fastqc.html)

**Per Base Sequence Quality** Looking at the first plot, we can clearly see this data is
not as high quality as the one we have been exploring ourselves.

**Per Tile Sequence Quality** Some physical artefacts are visible & some tiles seem to
be consistently lower quality. Whichever approach we take to cleaning the data will more
than likely account for any of these artefacts. Sometimes it’s just helpful to know where a
problem has arisen.

**Overrepresented Sequences** Head to this section of the report & scan down the
list. Unlike our sample data, there seem to be a lot of enriched sequences of unknown
origin. There is one hit to an Illumina adapter sequence, so we know at least one of the
contaminants in the data. Note that some of these sequences are the same as others on
the list, just shifted one or two base pairs. A possible source of this may have been non-random fragmentation.


Interpreting the various sections of the report can take time & experience.
A description of each of the sections [is available from the fastqc authors](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) which can be very helpful as you're finding your way.

Another interesting report is available at http://www.bioinformatics.babraham.ac.uk/projects/fastqc/RNA-Seq_fastqc.html.
Whilst the quality scores generally look pretty good for this one, see if you can find a point of interest in this data.
This is a good example, of why just skimming the first plot may not be such a good idea.

## Working With Complete Datasets

In our dataset of two samples it is quite easy to think about the whole experiment & assess the overall quality.

*What about if we had 100 samples?*

Each .zip archive contains text files with the information which can easily be parsed into an overall summary.
We could write a script to extract this information if we had the time.
However, some members of the Bioinformatics Hub have been writing an `R` package to help with this, which is available from https://github.com/UofABioinformaticsHub/ngsReports.

We'll publish this soon and using the package is beyond the scope of today.
However, we've included a [sample report](../data/ngsReports_Fastqc) of a dataset summarised using heatmaps.
This is simply the default report produced and the package is capable of exploring large datasets relatively easily.
Have a look at [this report](../data/ngsReports_Fastqc) and see if you can understand any of the plots.
Call an instructor over if you have any questions.
