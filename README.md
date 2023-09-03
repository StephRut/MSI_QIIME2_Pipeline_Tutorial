# MSI QIIME2 Pipeline Tutorial
This tutorial will be based on QIIME2 Version 2018.11. Commands and formats of files may change depending on the version of QIIME2 you are using.

_Documentation for QIIME2 Ver. 2018.11 can be found [here](https://docs.qiime2.org/2018.11/index.html)._

## Step 1: Software Requirements
  1) File transfer Software like FileZilla (Cyberduck, etc.)
  2) Text Editor like Atom, Notepad++
  3) Terminal such as Command Prompt, Windows Powershell
  4) Citrix to connect to servers else where(Cisco Anyconnect, etc.)
  5) Excel (GoogleSheets, etc.)
link to what 16s v4 ITS ITS2 is

## Step 2: Connect to MSI Server
**For Beginners, use a linux [Cheat Sheet](https://phoenixnap.com/kb/wp-content/uploads/2022/03/linux-commands-cheat-sheet-pnap.pdf)** 

 _Take note of File Commands and Directory Navigation_

 ```ssh``` into your server that you will be using for this tutorial

Locate your raw read files (look for files ending in .fastq or .fastq.gz) and copy the directory (folder) containing your reads into a new directory

```shell
cp -r project_1 ~/project_1
```
Here we are copying 'project_1' directory into a new directory named 'project_1' in our home directory.

Next, create a new directory for analysis of your data
   
```shell
mkdir project_1_analysis
```
    
6)  find [batch files](https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/blob/main/Batch%20Script.md) and Slurm scheduler
7)  link to MSI OnDemand
## Step 3: Creating a Manifest File
_Format Dependent on QIIME Version Used_

A manifest file is a file that maps each raw read file to a sample-id. Essentially, the manifest file associates all raw reads together and gives each read a ID.
Your manifest file will have 3 columns, with the headers: **'sample-id'**, **'absolute-filepath'**, and **'direction'**. Manifest files can be made in whatever program you feel most comfortable with, as long as you can save the file as a .csv. For an more information on creating a manifest file in Excel, click [here](url).
               
  _NOTE: Ensure that your column headers are spelled correctly, otherwise you may run into errors later on!_

The sample-ids can be whatever you want them to be so long as they are unique from each other (except when you have forward and reverse reads, in which case there should be two of each sample-id).
The absolute-filepath is the full path beginning from the root and ending with the directory/file you are specifying. No matter which directory you are in, an absolute filepath will always specify the same directory/file. In MSI, the absolute filepath will begin with ```/home```.

Ex: ```/home/absolute/filepath/project_1```

To determine what your absolute filepath will be, use the ```pwd``` command. The direction of the read will be either 'forward' or 'reverse'. Again, spelling matters! Check to see if your raw reads have both R1 and R2 in their names. If so, you have both forward and reverse reads or paired-end data.
R1 represents the forward read, and R2 represents the reverse.

How your Manifest file in Excel should look:


<img width="488" alt="Manifest_Example_Screenshot" src="https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/assets/125623174/ef7e8705-0fd5-4eaa-9ce2-a5adde515e63">

Before uploading your final Manifest file to QIIME2, ensure that your file looks like [this](https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/blob/main/Manifest_Example), with no extra spaces or blanks between columns. Extra blanks/spaces will lead to downstream errors. This tutorial will be working with paired-end data.

## Step 4: Load QIIME2 on MSI
Load QIIME2 on your remote server.
```shell
module load qiime2/2018.11
```
## Step 5: Import Manifest
Open up Filezilla (or other file sharing software) and connect to your desired server/remote site. Locate your Manifest file and transfer it over to the remote server in the directory you made previously for analysis. 

Next, convert your Manifest file into a QZA (QIIME 2 Artifact) format using the ```qiime tools import``` command.
```
qiime tools import \
--type SampleData[PairedEndSequencesWithQuality] \
--input-path Manifest_Example.csv \
--output-path demux.qza \
--input-format PairedEndFastqManifestPhred33
```
Learn more about the ```qiime tools import``` command [here](url).

Visualize a summary of your QZA file of demultiplexed sequences by converting it into a QZV (QIIME 2 Visualization) file. 

```
qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv
```
Learn more about the ```qiime demux summarize``` command [here](url).

Download the output QZV file onto your computer and upload the file into [QIIME 2 View](https://view.qiime2.org/). This visualization is similar to the FastQC reports, except it takes into account all of your raw reads when determining the quality scores. 

<img width="1007" alt="Demux Visualization Screenshot" src="https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/assets/125623174/d5e3d6ba-3cc9-4d8d-a81e-5e465ce7d5b3">

Try to get familiar with the interactive quality plots in this visualization.

## Step : Analyze FastQC

Analyzing the quality of your raw data can be crucial during the QIIME2 pipeline as poor quality reads might necessitate tweaking the steps you take in the pipeline. If you have a FastQC report on your samples, please take the time to assess the overall quality of your reads. 

[FastQC documentation](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), [Good FastQC data](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/good_sequence_short_fastqc.html), [Bad FastQC data](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/bad_sequence_fastqc.html)

Generally, the quality score of a base will decrease as the base position increases. It's also common to see the first 5-10 bases have a lower quality score than the bases following. 

FOR MSI USERS: Within the directory that you found your raw Fastq files in, follow the filepath ```analysis/illumina-basicQC```. Look over the file "report.html" This will take you to an Illumina BasicQC report. You should also view a few of the FastQC reports generated, which can be found by following ```analysis/illumina-basicQC/fastqc``` again, starting from the directory containing your raw Fastqc files.

For more information on FastQC files, view the [FastQC Basics](url) file.

## Step 6: Trim Primers
For 16S data, the process of trimming primmers is fairly straight forward. For ITS data, please view the [Trimming ITS Primer](url) file. The forward and reverse primers are the same sequences that were specified for PCR. Oftentimes, the V4 region of 16S rRNA is used, which can be isolated using the 515F (Parada) - 806R (Apprill) primer pair with forward primer: GTGYCAGCMGCCGCGGTAA and reverse primer: GGACTACNVGGGTWTCTAAT
```
qiime cutadapt trim-paired \
--i-demultiplexed-sequences demux.qza \
--p-front-f GTGYCAGCMGCCGCGGTAA \
--p-front-r GGACTACNVGGGTWTCTAAT \
--p-cores 20 \
--o-trimmed-sequences trimmed-seqs.qza 
```
Learn more about the ```qiime cutadapt trim-paired``` command [here](url). 

Next, create a visualization summary of your trimmed sequences/reads.

```
qiime demux summarize \
--i-data trimmed-seqs.qza \
--o-visualization trimmed-seqs.qzv
```

## Step : Visualize Trimmed Reads

Upload your trimmed sequences QZV file to QIIME 2 View to look at the quality of your reads. This visualization should look similar to the previous QIIME 2 visualization. The key difference is that your sequences should now be shorter in length. 

<img width="758" alt="before and after trimming length" src="https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/assets/125623174/971e2434-1e2e-4c69-8db5-00b27124f859">

In 16s data, the quality of your reads will determine the lengths where you should trim and truncate the sequences. In ITS data, it is generally recommended to filter your data using different metrics, see Filtering Step. 

## Step 7: Filter the Sequences
If you are filtering ITS data, see [Filtering ITS](url) page.

For 16s rRNA data, especcially paired-end reads, it is important to know the length of the sequences you are targeting. **(LOOK FOR ARTICLE GIVING ROUGH LENGTH OF V4 REGION HERE)**
If you trim paired-end reads too short, then there might not be enough of an overlap to merge the reads. On the other hand, if the quality scores are too low on the end of your reads, the data is less likely to be an accurate representation of what species are present in your sample. If your paired-end reads must be trimmed short due to low quality reads towards the end of the sequence, it might be best to analyze the forwards sequences as single-end reads instead.

The key here is to find the sweet spot where you keep as much data as you can, without sacrificing the quality of the data.

Ideally, we want to keep the median quality score of the sequences equal to or above 30 or q30. Meaning when we look at our last QIIME 2 Visualization, we will trim right before the first location in the sequence where the median is < 30. 

 7) explain what a quality score is 30

ITS:
8) explain EE
9) explain p-trunc-q

## Step :
visualize the dada stats... make sure you aren't loosing too many reads ideally 70% recovery

## Step 8: Training the Classifier
 1) Download the Classifier from either green genes or silva 16s or Unite ITS
 2) Link both of them on the page
 3) make characters uppercase to avoid downstream errors
 4) 99 sequences .fasta to .qza
 5) 99 taxonomy .txt to .qza
 6) train classifier code
 7) Learn the classifier (i believe 16s you trim to V4 region) but in ITS its best not to do trimming
## Step 9: Building ASV Table with Taxonomy
 1) create taxa feature table (ASV)
 2) convert taxa table into .txt
 3) open in excel
 4) analysis

    [link](https://github.com/StephRut/MSI_QIIME2_Pipeline_Tutorial/blob/main/google28c0f62a2c89c942.html)

