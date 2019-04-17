* TOC
{:toc}

# Adapter and quality trimming of NGS data

Once we have inspected our data and have an idea of how accurate our reads are, as well as any other technical issues that may be within the data, we may need to trim or filter the reads to make sure we are aligning or analysing sequences that accurately represent our source material.  As we’ve noticed, the quality of reads commonly drops off towards the end of the reads, and dealing with this behaviour can be an important part of most processing pipelines. Sometimes we will require reads of identical lengths for our downstream analysis, whilst other times we can use reads of varying lengths. The data cleaning steps we choose for our own analysis will inevitably be influenced by our downstream requirements.

## The Basic Workflow

Data cleaning and pre-processing can involve many steps, and today we will use the basic work-flow as outlined below.
Each analysis is slightly different so some steps may or may not be required for your own data, however many workflows do have a little overlap, and some pipelines (_e.g._ *Stacks*) may even perform some of these steps for you.

*A basic workflow is:*

1. **Remove Adapters and Quality Trim** (`AdapterRemoval`)
2. **Run FastQC** on trimmed reads.
3. **Alignment** to a reference (`bwa`, `bowtie2`, `STAR`)
4. **Post-alignment QC** (`picard markDuplicates`, `IGV`)


## Removal of Low Quality Bases and Adapters

Adapter removal is an important step in many sequencing projects, mainly projects associated with DNA/RNA inserts which are shorter than the sequencing read lengths.
A good example of this would be an experiment where the target molecule is small non-coding RNAs.
As these are generally between 19-35bp, which is shorter than the shortest read length provided by Illumina sequencing machines, all reads containing a target molecule will also contain adapters.
In addition, when we size select our initial fragments, we select a range of fragment sizes and some are bound to be shorter than our read length.
Therefore it is important to trim adapters accurately to ensure that the genome mapping and other downstream analyses are accurate.

In the early years of NGS data, we would run multiple steps to remove both adapters, low quality bases (near the ends of reads) and reads which have overall lower quality scores.
Today's trimming algorithms have become better at removing low-quality bases and the same time as removing adapters.
The tool we'll use for this step today is [`AdapterRemoval`](https://buildmedia.readthedocs.org/media/pdf/adapterremoval/latest/adapterremoval.pdf).

Now we can trim the raw data using the Illumina Nextera paired-end adapters obtained from [this website](https://support.illumina.com/bulletins/2016/12/what-sequences-do-i-use-for-adapter-trimming.html)
These are commonly used in Illumina projects.

**Before we perform adapter trimming, look at the following code.**

```
cd ~/agingRnaSeq/
AdapterRemoval \
	--file1 0_rawData/fastq/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz \
	--output1 1_trimmedData/fastq/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz \
	--discarded 1_trimmedData/fastq/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.discarded.gz \
	--minlength 50 \
	--threads 2 \
	--trimns \
	--trimqualities \
	--minquality 20 \
	--gzip \
	--settings 1_trimmedData/log/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz.settings
```

#### Questions
{:.no_toc}
*1. What do the options* `--minlength 50` *and* `--minquality 20` *specify in the above? Do you think these choices were reasonable?*
*2. Notice the adapter sequence wasn't specified anywhere. Did we miss an important setting?*
*3. What do you expect to find in the file specified using the `--discarded` option?*
*4. What do you expect to find in the file specified using the `--settings` option?

Run the above code by pasting into your terminal.
Did you guess correctly?

The `AdapterRemoval` tool can be made to output information about the trimming process to a file.
In the above we wrote this output to a "settings" file using the `--settings` option to output this to the `1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz.settings` file.
Let's have a look in the file to check the output.

```
less 1_trimmedData/log/1_non_mutant_Q96_K97del_6mth_10_03_2016_S1_fem_R1.fq.gz.settings
```

As these were a good initial sample, it's not surprising that we didn't lose many sequences.
Notice that many reads were trimmed (`Number of well aligned reads`), but were still long enough and high enough quality to be retained.

### Trimming using a script

In the above, we manually ran the process on an individual file and manually specified key information about where to write various output files.
In the real world, this is not really a viable option and we'll need to write a script to trim all of our samples.

Before we write this script, let's think about what will be involved.

1 - We'll need to provide a list of input fastq files to work on
2 - We'll need to specify different output files
3 - Some parameters will be constant, whilst others will change for every file

In following excerpts, we'll (hopefully) give you all the clues you need to complete this script, particularly when you look at the actual command we gave to AdapterRemoval above.

**How do we find our files?**

Copy the following into a new file in your bash folder by entering the command `nano bash/removeAdapters.sh` (make sure you're in the main project directory first by entering `cd ~/agingRnaSeq`)

```
#!/bin/bash

# Setup our input directory, then look for the files we need
INDIR=~/agingRnaSeq/0_rawData/fastq
INFILES=$(ls ${INDIR}/*fq.gz)

# Check that we have found the correct files
for f in ${INFILES}
do
    echo "Found ${f}"
done
```

Exit `nano` and save by entering `Ctrl+x` answering `y` when asked if you'd like to save changes.
Let's run the script using `bash bash/removeAdapters.sh`

**Did you see the files you expected returned?**

Try changing the second to last line to be the following
```
    echo "Found $(basename ${f})" 
```

**What was the difference?**

Now that we've figured how to find our input files and remove the input directory from their names, let's see if we can create new names for output files.
Change your script to be the following.

```
#!/bin/bash

# Setup our input directory, then look for the files we need
INDIR=~/agingRnaSeq/0_rawData/fastq
INFILES=$(ls ${INDIR}/*fq.gz)

# Define our output parent directory
OUTDIR=~/agingRnaSeq/1_trimmedData

# Check our output directory exists
if [ ! -d "$OUTDIR" ]; then
    # If the directory doesn't exist, exit with a message and an error
    echo "$OUTDIR does not exist"
    exit 1
fi

# Check that we have found the correct files
for f in ${INFILES}
do
    echo "Found ${f}"
    OUTFILE=${OUTDIR}/fastq/$(basename ${f})
    echo "Trimmed reads will be written to ${OUTFILE}"
done
```

Now you've seen how specify the output file, by using the input file we're nearly there.
**What else do you think we need to specify?**

Add the following lines to your script on a new blank line immediately before our final `done` command.
If you're having trouble with any of this please call a tutor over too!

```
    DISCARDED=${OUTDIR}/fastq/$(basename ${f%.fq.gz}.discarded.gz)
    echo "Discarded reads will be written to ${DISCARDED}"
```

Note that in the above we used the `%` sign like a pair of scissors that snipped off the suffix `fq.gz`, and we then replaced it with the new suffix `discarded.gz`.
Clearly, the next file we need to specify will by our log or settings file.

```
    SETTINGS=${OUTDIR}/log/$(basename ${f%.fq.gz}.settings)
    echo "AdapterRemoval settings file will be written to ${SETTINGS}"
```

Now we have all the information we need to run `AdapterRemoval` on every input file!
We'll leave it up to you to write the complete script, but please call for help when you need it.

**Can you think of anything important we skipped over in the above?**<details>
We didn't check for the existence of the output directories for the trimmed fastq files, or for the settings files
</details>

The complete script should run in about 10 minutes.

### FastQC

Now that we've run `AdapterRemoval` on our complete set of files, we'll need to check what impact it's had on our libraries.
An intelligent approach would have been to include a call to `FastQC` as the final step in the above script.
If you think you're ahead, add the following line to your script and run it all again.
Otherwise, just run this command interactively

```
fastqc -t 2 -o 1_trimmedData/FastQC/ 1_trimmedData/fastq/*fq.gz
```

Once this completes, use RStudio to manually inspect a report, then use the function `writeHtmlReport` from the R package `ngsReports` to write the default summary across all files.
You can also use the ShinyApp `fastqcShiny` to inspect these interactively if you'd like.

By comparing this new report to the original report, **are you happy with the improvements to the data?**

