# Instructions for Computer Setup
{:.no_toc}

* TOC
{:toc}

For the *Introduction to Bash* workshops we ran everything on our own machines so we would all know how to access `bash` and all of the important tools after the workshop.
Unfortunately, some of the tools we'll come across today are only available on Linux or OSX so we'll use a Virtual Machine (VM) which we have already placed all the data on and installed all the tools.

If you'd like to try installing all the tools locally and running the session please see below, however this may be a risky  exercise.
If you're confident installing software, then do feel free as it may be a beneficial experience despite the challenges.

## Connecting to the VM

To connect to your VM, we'll use the [X2Go Client Software](https://wiki.x2go.org/doku.php/doc:installation:x2goclient).
If you don't already have this installed, please do so.
Once this is installed:

1. Open the X2Go Software
2. Enter "Introduction to NGS Data" or something similar in the **Session name** field at the top
3. Enter your assigned IP address in the **Host** field
4. Enter `trainee` as the **Login**
5. Under **Session Type**, use the drop-down menu to select `XFCE`
6. Enter `OK`

This **will not** log you in, but will have setup the "session" and provided a login link in the top-right of the X2Go program window.
Once you've figure out what we mean, click this link and you will be taken to a login which requires you to enter your password.
Enter the password `trainee` and you should be connected to your VM.

At this point, you should have a desktop visible on the VM which look something like this:

![](../images/VM_Desktop.png)

From here, you will be able to complete all of today's material.
If you're a bit ambitious and would like to use your own terminal connected to the VM, you should be able to `ssh` in using the same login information as above.
If you don't know what we mean, then just carry on using X2Go.

## Installing Software Locally

**NB: For the confident people only**

This is only possible if you're running a Mac or Linux system.
However, these installation instructions are written for Mac, and have **not** been tested.
If you're running a Linux OS, change the first line to download the correct installation script (i.e. https://repo.continuum.io/miniconda/Miniconda3-3.7.0-Linux-x86_64.sh).
Everything else should be exactly the same.

First you'll need to minconda (taken from https://conda.io/docs/user-guide/install/macos.html)

```
wget https://repo.continuum.io/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -p $HOME/miniconda
```

Now we'll remove any installation channels that we don't need, and add the ones that we do

```
conda config --remove channels r
conda config --add channels defaults
conda config --add channels conda-forge
conda config --add channels bioconda
```

Finally, we'll just install the packages that we need.
Note that this may take a while.

```
conda install --yes bwa sambamba samtools igv bedtools fastqc picard freebayes cutadapt bcftools
cd ~/Downloads
git clone https://github.com/najoshi/sabre.git
cd sabre
make
mv sabre ~/miniconda/bin
cd ~
```

Now we'll need to setup the data and directories on your local machine.
Firstly we'll get the WGS data:

```
mkdir -p ~/WGS/01_rawData/fastq
cd ~/WGS/01_rawData/fastq
wget -c https://universityofadelaide.box.com/shared/static/23r1szeg3z3wtzcs1my2szw63w8zv7ip.gz -O subData.tar.gz
tar xzvf subData.tar.gz
rm subData.tar.gz
mv chr* ~/WGS
```

Now we'll get the multiplexed data for that section:

```
cd ~
mkdir -p ~/multiplexed/01_rawData/fastq
cd ~/multiplexed/01_rawData/fastq
wget -c https://universityofadelaide.box.com/shared/static/sdgu5v4m0i63mfybkl3x81dmgwyaikr2.gz -O multiplexed.tar.gz
tar xzvf multiplexed.tar.gz
rm multiplexed.tar.gz
mv barcodes_R1.txt ~/multiplexed
```

Now you should be good to go
