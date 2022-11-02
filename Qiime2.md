# Gracilaria 2022: qiime2
---UPDATED October 2022 


###### tags: `gracilaria`, `qiime2`




![](https://i.imgur.com/s3VM4Gv.png)

https://docs.qiime2.org/2021.8/tutorials/overview/ 

For an overview of amplicon sequencing, you can consult this book chapter for an intro on DNA extraction, PCR, sequencing technologies and data analysis: [Yates, 2016](https://drive.google.com/file/d/1zKUlOYZefoP3ZAGHv32cb7iROuHm3tgd/view?usp=sharing)

[TOC]



## Getting your terminal ready

[**Here is a link to the MAE Unix tutorial if you need a quick review**](https://jbpc.mbl.edu/unix-tutorial/MAE2020.html)

**are you running in a screen session?**
Yes? Great. 
No? You'll want to be.  [Here is a link to the full set of instructions](https://hackmd.io/Y3AabwWoTpObumF4eYb37g?view#screen) 

### Usage of screen

navigate to the folder where you will run your analysis

1. Get a list of active screen sessions: `screen -r`
2. To connect to an existing screen session `screen -r sessionID`
3. To connect to screen session `screen -r sessionID`
4. To create and name a new session `screen -S sessionID`
5. To generate a file to log specific parts of the session:
Once in a screen session, press the control key and `a` at the same time, then `:` (we write this as `ctrl-a + :`).
A line of text will appear on the botton of the terminal. 
Type `logfile NameForLog.txt` and then press enter (twice) to create the log file.  
Then, if you want to log something, use `ctrl-a + h`.  A line saying creating file NAMEforlog.txt will appear, press enter and everything you type will be recorded. To stop recording, just use`ctrl-a + h` againg to turn off loggin. To turn it back on, `ctrl-a + h` (it will append to your log file rather than overwrite).
6. To detach the screen session (which is like leaving it running but returning to your non-screen prompt) `ctrl-a + d` 
7. To end a screen type `exit`
8. If that doesn't work, to kill a screen session `ctrl-a + k` 



# Qiime?

QIIME (Quantitative Insights Into Microbial Ecology) 



To start qiime2:  

```
module purge
    module load qiime2/2022.8-conda
    conda activate /bioware/qiime2-2022.8-conda
  
#If it generates a complaint that your environment isn't set up
properly, you can resolve that with:
   
        conda init bash
    . ~/.bashrc
    conda activate /bioware/qiime2-2022.8-conda
    
```

To exit qiime2:


```
conda deactivate

```

This deactivates the conda-qiime environment and you will see that you no longer have '(qiime)' preceding your prompt.

# Importing data into Qiime2 and generating feature tables

This flowchart describes all demultiplexing steps that are currently possible in QIIME 2, depending on the type of raw data you have imported. Usually only one of the different demultiplexing actions available in q2-demux or q2-cutadapt will be applicable for your data, and that is all you will need. In this tutorial we will use demultiplexed paired-end reads stored in fastq files. We will merge the reads and denoise the data using Dada2. 

![](https://i.imgur.com/SwxRkwA.png)


## Data import

:::success
qiime tools import \
&nbsp;    --type 'SampleData[PairedEndSequencesWithQuality]' \
&nbsp;    --input-path pe-33-manifest.txt \
&nbsp;    --output-path paired-end-demux.qza \
&nbsp;    --input-format PairedEndFastqManifestPhred33V2 
:::

Run in terminal

```
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path pe-33-manifest.txt --output-path paired-end-demux.qza  --input-format PairedEndFastqManifestPhred33V2

```

Now we are going to generate a human readable summary (we will do this after many steps, first we compute the data, then we summarize)
:::success
qiime demux summarize \
&nbsp;    --i-data paired-end-demux.qza  \
&nbsp;    --o-visualization paired-end-demux.qzv
:::

Run in the terminal
```
qiime demux summarize --i-data paired-end-demux.qza --o-visualization paired-end-demux.qzv
```
![](https://i.imgur.com/kdVtDX3.png)

Forward

![](https://i.imgur.com/tOXfOSA.png)

Reverse

![](https://i.imgur.com/cLAUcBm.png)

REverse seems to go bad around 220. 
  
## Filtering and Clustering

![](https://i.imgur.com/85h8SCt.png)


DADA2 to 
* remove low quality reads and pairs
* merge
* cluster
* remove chimeras
for more info see here: https://docs.qiime2.org/2020.6/plugins/available/dada2/denoise-paired/ 


:::success
qiime dada2 denoise-paired \
&nbsp;    --i-demultiplexed-seqs paired-end-demux.qza \
&nbsp;    --p-trim-left-f 23 \
&nbsp;    --p-trim-right-f 18 \
&nbsp;    --p-trunc-len-f 235 \
&nbsp;    --p-trunc-len-r 210 \
&nbsp;    --o-table table.qza \
&nbsp;    **--p-n-threads 25** \
&nbsp;    --o-representative-sequences rep-seqs.qza \
&nbsp;    --o-denoising-stats denoising-stats.qza \
&nbsp;    --verbose\
:::

Run in the terminal

```
qiime dada2 denoise-paired  --i-demultiplexed-seqs paired-end-demux.qza --p-trim-left-f 23   --p-trim-left-r 18   --p-trunc-len-f 235  --p-trunc-len-r 220  --o-table table.qza  --p-n-threads 25 --o-representative-sequences rep-seqs.qza   --o-denoising-stats denoising-stats.qza --verbose

qiime feature-table summarize  --i-table table.qza  --o-visualization table.qzv  --m-sample-metadata-file sample-metadata.tsv

qiime feature-table tabulate-seqs   --i-data rep-seqs.qza  --o-visualization rep-seqs.qzv

qiime metadata tabulate   --m-input-file denoising-stats.qza  --o-visualization denoising-stats.qzv

```
Only 60% passed quality filter. Merging and chimeric removal retained average of 45%. 

Try with different trimming. 
  ```
qiime dada2 denoise-paired  --i-demultiplexed-seqs paired-end-demux.qza --p-trim-left-f 23   --p-trim-left-r 18   --p-trunc-len-f 235  --p-trunc-len-r 212  --o-table table2.qza  --p-n-threads 25 --o-representative-sequences rep-seqs2.qza   --o-denoising-stats denoising-stats2.qza --verbose

qiime feature-table summarize  --i-table table2.qza  --o-visualization table2.qzv  --m-sample-metadata-file sample-metadata.tsv

qiime feature-table tabulate-seqs   --i-data rep-seqs2.qza  --o-visualization rep-seqs2.qzv

qiime metadata tabulate   --m-input-file denoising-stats2.qza  --o-visualization denoising-stats2.qzv

```
similar results. 
 Use the second triming conditions as we gained .5%. 



# Taxonomic assignment

![](https://i.imgur.com/4GxuhNm.png)


## Adquiring databases: region V4V5


**very important** your database has to match the version of your qiime2 installation (in this case 2021.8)


There is a variety of databases we can use for taxonomical classification.See [here](https://docs.qiime2.org/2021.8/data-resources/). Today, we will be using Silva. 
```
wget  -O "silva-138-99-515-806-nb-classifier.qza" "https://data.qiime2.org/2021.8/common/silva-138-99-515-806-nb-classifier.qza"
```
## Classifying the sequences



**Using the database for V4V5 region**
:::success
qiime feature-classifier classify-sklearn  \
&nbsp;    --i-classifier silva-138-99-515-806-nb-classifier.qza   \
&nbsp;    --i-reads rep-seqs.qza   \
&nbsp;    --o-classification taxonomy.qza --p-n-jobs 4
:::
(this step will also take some time)
In the terminal
```
qiime feature-classifier classify-sklearn  --i-classifier silva-138-99-515-806-nb-classifier.qza  --i-reads rep-seqs.qza  --o-classification taxonomy.qza --p-n-jobs 4
```

and summarize

:::success
qiime metadata tabulate  \
&nbsp;    --m-input-file taxonomy.qza  \
&nbsp;    --o-visualization taxonomy.qzv 
:::

In the terminal
```
qiime metadata tabulate  --m-input-file taxonomy.qza --o-visualization taxonomy.qzv
```


## Visualizing Taxonomy using bar plots.

One of the most popular ways of visulaizing the taxonomical composition of a sample is using blar plot. This has the advantage of allowing us to compare many samples at one. This type of represeantion is useful to visualize the most abundant groups.  

:::success
qiime taxa barplot \
&nbsp;    --i-table table.qza \
&nbsp;    --i-taxonomy taxonomy.qza \
&nbsp;    --m-metadata-file sample-metadata.tsv \
&nbsp;    --o-visualization taxa-bar-plots.qzv

:::



```
qiime taxa barplot  --i-table table.qza --i-taxonomy taxonomy.qza --m-metadata-file sample-metadata.tsv  --o-visualization taxa-bar-plots.qzv
```
tax bar plots

![](https://i.imgur.com/wbHOvo3.png)


Eukaryote algae in the microbial community of gracilaria. 
![](https://i.imgur.com/Ps5cxW4.png)
 
**Export ASV table**

You might want to have your data in an annotated excel table. In this tutorial we are going to create a folder and an excel compatible file for each table we generate. You can export the data as follows. 
:::success
qiime tools export   \
&nbsp;    --input-path table.qza  \
&nbsp;    --output-path ASV-table
:::

```
qiime tools export  --input-path table.qza   --output-path ASV-table

```

Convert the exported table from biom format to tsv

:::success
biom convert \
&nbsp;    -i ASV-table/feature-table.biom \
&nbsp;    -o ASV-table/feature-table.tsv \
&nbsp;    --to-tsv
:::

```
biom convert -i ASV-table/feature-table.biom -o ASV-table/feature-table.tsv --to-tsv

```


# Using taxonomical information to remove eukaryotic sequences


:::success
qiime taxa filter-table   \
&nbsp;    --i-table table.qza   \
&nbsp;    --i-taxonomy taxonomy.qza   \
&nbsp;    --p-exclude mitochondria,chloroplast   \
&nbsp;    --o-filtered-table table-no-mitochondria-no-chloroplast.qza
:::
```
qiime taxa filter-table --i-table table.qza  --i-taxonomy taxonomy.qza --p-exclude mitochondria,chloroplast  --o-filtered-table table-no-mitochondria-no-chloroplast.qza
```


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::
```
qiime feature-table summarize  --i-table table-no-mitochondria-no-chloroplast.qza  --o-visualization table-no-mitochondria-no-chloroplast.qzv  --m-sample-metadata-file sample-metadata.tsv
  
```


Now let's visualize the new table without chloroplast and mitochondria. 
We will use the same scripts we used after dada2, just modifing the inputa table, create summary tables of clustered sequences.

Type:


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-no-mitochondria-no-chloroplast.qza  --o-visualization table-no-mitochondria-no-chloroplast.qzv   --m-sample-metadata-file sample-metadata.tsv
```
#### Identification of eukaryotic sequences
Extract the eukaryotic sequences (chloroplast, mitochondria) from file Taxonomy_table.tsv.(generate from qiime2 taxonomy table). opy all ASV IDs into file Euk_ASV.txt. 

Use 

```
grep -w -A 2 -f Euk_ASV.txt sequences.fasta --no-group-separator >EuK_ASV.fasta

blastn -db nt -query EuK_ASV.fasta -num_threads 12 -max_target_seqs 5 -outfmt '6 qseqid sseqid evalue bitscore sgi sacc staxids sscinames scomnames stitle' -out EuK_ASV.fasta.out 

```

get frequency data from the table.qvz (feature detail). Copy into a text and extract in R the frequencies

```
#in R

merged.data <- merge(Euk_ASV2, ASV_frequencies, by="ASV")
write.csv(merged.data, file = "Euk_frequencies.csv", row.names = TRUE, col.names = TRUE)
```


## Generate new bar plots after removal of eukaryotic sequences
 
 :::success
 qiime taxa barplot   \
 &nbsp; --i-table table-no-mitochondria-no-chloroplast.qza  \
 &nbsp; --i-taxonomy taxonomy.qza   \
 &nbsp; --m-metadata-file sample-metadata.tsv  \
 &nbsp; --o-visualization taxa-bar-plots-no-mitochondria-no-chloroplast.qzv
 :::
 
 ``` 
 qiime taxa barplot   --i-table table-no-mitochondria-no-chloroplast.qza   --i-taxonomy taxonomy.qza   --m-metadata-file sample-metadata.tsv   --o-visualization taxa-bar-plots-no-mitochondria-no-chloroplast.qzv
 
 ```
 
 
![](https://i.imgur.com/EfcmAi1.png)


**Export ASV table**

Now, we have a filtered ASV table that can be used in other analysis, e.g.  differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-no-mitochondria-no-chloroplast.qza  --output-path ASV-table-no-mitochondria-no-chloroplast
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-no-mitochondria-no-chloroplast/feature-table.biom -o ASV-table-no-mitochondria-no-chloroplast/feature-table.tsv --to-tsv
```

# Quality controls and sample selection

## Rarefaction Analysis 

**Now that we have a bacterial dataset, we will examine how thoroughly we have "sampled our sample"**

Our original DNA extractions represent the population of microbes in the environment. We know that different taxa are present in different abundances in any sample. If our sequencing effort has not been enough, we might be missing out those taxa that are not very abundant (rare biosphere). How do we know id we may or may not have sequenced our samples deeply enough to detect everything that is in the sample?.
We test whether we have reached saturation in taxa discovery, that is, we have identified (most of) the organisms in that sample. 
 
## Rarefaction curve for visualization of taxa discovery. 

We are going to visualize whether your taxa discovery for each sample has reached saturation given your sequencing effort using a rarefaction curve. We will also be able to determine at what sequencing depth saturation happened for each sample. 
The alpha rarefaction command randomly subsamples your dataset at n = 1000, n = 2000, n = 3000... n = 50,000 sequences, and it plots the # of unique microbial taxa at each sequencing depth, generating a rarefaction curve. 

*Here we will use a --p-max-depth of 65000. If this too high, the program will tell you that the maximum number of features is **X**. Use that number as --p-max-depth instead.*

:::success
qiime diversity alpha-rarefaction  \
&nbsp; --i-table table-no-mitochondria-no-chloroplast.qza \
&nbsp; --p-max-depth 65000  \
&nbsp; --m-metadata-file sample-metadata.tsv  \
&nbsp; --o-visualization alpha-rarefaction.qzv
:::

```
qiime diversity alpha-rarefaction   --i-table table-no-mitochondria-no-chloroplast.qza --p-max-depth 65000  --m-metadata-file sample-metadata.tsv  --o-visualization alpha-rarefaction.qzv

```
you can also run a second curve with smaller sequencing depth to focus on the beginning of the curve

```
qiime diversity alpha-rarefaction   --i-table table-no-mitochondria-no-chloroplast.qza --p-max-depth 8000  --m-metadata-file sample-metadata.tsv  --o-visualization alpha-rarefaction-2.qzv
```

If your visualization looks something like this then all samples were sequenced to a depth sufficient to sample all (or almost all) of the diversity:

![](https://i.imgur.com/KT8kR6u.png)

If you have a sample that are not reaching or approaching an asymptope then you (1) could consider investing in deeper sequencing of the sample or (2) you can  remove the undersequenced sample. 
You dont need to reanalyze all our data, we can remove a sample by fitlering it out using the metadata file (next section). 

Now, use the graphs and table files to check if all your samples have reached saturation. If not, identify those samples that should be filtered our from the analysis.

![](https://i.imgur.com/osCopOU.png)

![](https://i.imgur.com/bJSX5K2.png)


## Metadata-based filtering

You might want to focus at this time in a subset of samples, or remove those samples that are low quality. To do so, we can create new files and only include the samples that you want to analyze. To do this, we are going to filter samples. 

To filter samples based on quality: 
One of the easiest ways of doing this is to modify the metadata file adding a column called keep. Identify the samples you want to keep with the category "yes", and those to be filtered with "no".


![](https://i.imgur.com/WJRoTO5.png)



`--p-where "[keep]='yes'"`


**From this point onwards, "table-no-mitochondria-no-chloroplast.qza" becomes "table-filtered-no-mitochondria-no-chloroplast.qza"!and only includes the subset of samples of interest**

:::success
qiime feature-table filter-samples  \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --m-metadata-file sample-metadata.tsv   \
&nbsp;    --p-where "[keep]='yes'"   \
&nbsp;    --o-filtered-table table-filtered-no-mitochondria-no-chloroplast.qza
    
:::

```
qiime feature-table filter-samples  --i-table table-no-mitochondria-no-chloroplast.qza  --m-metadata-file sample-metadata.tsv   --p-where "[keep]='yes'"  --o-filtered-table table-filtered-no-mitochondria-no-chloroplast.qza

```

Now let's visualize the new filtered table
We will use the same scripts we used after dada2, just modifing the input table to create summary tables of sequences.

Type:


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-filtered-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --o-visualization table-filtered-no-mitochondria-no-chloroplast.qzv   --m-sample-metadata-file sample-metadata.tsv
```

:::warning
** How to filter samples based in metadata information:**
You can select your samples based in any of the columns in the metadata. 
Maybe you only want the DNA samples, or the samples from the initial timepoint. 
You can use the same approach, just select the column from the metadata file and the variable. 
```
iime feature-table filter-samples  \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --m-metadata-file sample-metadata.tsv   \
&nbsp;    --p-where "[ColumnName]='ANYTHING_YOU_WANT'"  \
&nbsp;    --o-filtered-table table-ANYTHING_YOU_WANT-no-mitochondria-no-chloroplast.qza
 ```   

**If there are multiple values that should be retained from a single metadata column, the IN clause can be used to specify those values.** For example, the following command can be used to retain all samples with Ulva and Seawater.

```
qiime feature-table filter-samples    \
&nbsp;      --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;      --m-metadata-file sample-metadata.tsv   \
&nbsp;      --p-where "[ColumnName] IN ('ANYTHING_YOU_WANT01','ANYTHING_YOU_WANT02')"   \
&nbsp;      --o-filtered-table table-ANYTHING_YOU_WANT_01_02-no-mitochondria-no-chloroplast.qza

```
:::

## Generate final taxonomy graphs

Use the final filtered files to generate the bar plots. 
:::info
If you have filtered by metadata make sure you are calling the right files. Double check the table file names. In fact, for your convenience, I recommend you to rename the table files from `table-ANYTHING_YOU_WANT_01_02-no-mitochondria-no-chloroplast.qza` to `table-filtered-no-mitochondria-no-chloroplast.qza` as this is how the file will be called from now on.
:::

 
 :::success
 qiime taxa barplot   \
 &nbsp; --i-table table-filtered-no-mitochondria-no-chloroplast.qza  \
 &nbsp; --i-taxonomy taxonomy.qza   \
 &nbsp; --m-metadata-file sample-metadata.tsv  \
 &nbsp; --o-visualization taxa-bar-plots-filtered-no-mitochondria-no-chloroplast.qzv
 :::
 
 ``` 
 qiime taxa barplot   --i-table table-filtered-no-mitochondria-no-chloroplast.qza   --i-taxonomy taxonomy.qza   --m-metadata-file sample-metadata.tsv   --o-visualization taxa-bar-plots-filtered-no-mitochondria-no-chloroplast.qzv
 
 ```
 
 :::info
 **Can you see this graph?**
 What is the impact of the different treatments in the samples?
Does it matter if we look to who is present (DNA) or who is active (RNA)?

 
 ![](https://i.imgur.com/VpoxUeh.png)
 :::
 
**Export as ASV table**

ASV tables are useful for additional analyiss, e.g. for differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-filtered-no-mitochondria-no-chloroplast.qza  --output-path ASV-table-filtered-no-mitochondria-no-chloroplast
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-filtered-no-mitochondria-no-chloroplast/feature-table.biom -o ASV-table-filtered-no-mitochondria-no-chloroplast/feature-table.tsv --to-tsv
```

## Taxonomy-collapse 
Now that we have a definitive table without eukaryotic sequences and filtered of low quality samples, we can use this option to export data at the taxonomical rank of our choice.This simplified our dataset which is useful for visualization but also in some of the additional analysis we will run.  

https://docs.qiime2.org/2021.8/tutorials/pd-mice/taxonomic-classification-again

```
qiime taxa collapse  --i-table table-filtered-no-mitochondria-no-chloroplast.qza --i-taxonomy taxonomy.qza   --o-collapsed-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza --p-level 7
```
You can export the table in to excel for additional analysis.  
```
qiime tools export   --input-path table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --output-path table-collapsed-filtered-no-mitochondria-no-chloroplast-level7
```

Convert the exported table from biom format to tsv
```
biom convert -i table-collapsed-filtered-no-mitochondria-no-chloroplast-level7/feature-table.biom -o table-collapsed-filtered-no-mitochondria-no-chloroplast-level7/feature-table.tsv --to-tsv
```


# Part I. Statistical Analyses
In microbiome experiments, investigators frequently wonder about things like:

* How many different species/OTUs/ASVs are present in my samples?
* How much phylogenetic diversity is present in each sample?
* How similar/different are individual samples and groups of samples?
* What factors (e.g., pH, elevation, blood pressure, body site, or host species just to name a few examples) are associated with differences in microbial composition and biodiversity?

And more. These questions can be answered by alpha- and beta-diversity analyses. Alpha diversity measures the level of diversity within individual samples. Beta diversity measures the level of diversity or dissimilarity between samples. We can then use this information to statistically test whether alpha diversity is different between groups of samples (indicating, e.g., that those groups have more/less species richness) and whether beta diversity is greater between groups (indicating, e.g., that samples within a group are more similar to each other than those in another group, suggesting that membership within these groups is shaping the microbial composition of those samples).
![](https://i.imgur.com/0glFIdz.png)
## Alpha Diversity

There are many ways to express the observed alpha diversity of a sample and the estimated richness of the population based on the abundances of taxa (OTUs, ASVs) in the sample.  The [Shannon](https://en.wikipedia.org/wiki/Diversity_index#Shannon_index) and Simpson Diversity Indeces are two common ways of expressing the richness and relative abundance of taxa observed in the sample; Chao1 is a common nonparametric estimator total richness.  
To see a complete list of available metrics:
```
qiime diversity alpha --help
```
:::info
Additional information on 
[alpha and beta diversity metrics](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282)

:::

And each is defined [here](http://scikit-bio.org/docs/latest/generated/skbio.diversity.alpha).
e.g. 
**Shannon’s index**: Calculates richness and diversity using a natural logarithm. Accounts for both abundance and evenness of the taxa present

First we generate the diversity metric; for shannon index the syntax would be:



:::success
qiime diversity alpha \
&nbsp;   --i-table  table-filtered-no-mitochondria-no-chloroplast.qza \
&nbsp;   --p-metric shannon \
&nbsp;   --o-alpha-diversity ALPHA-shannon-vector.qza 
:::
```
qiime diversity alpha --i-table  table-filtered-no-mitochondria-no-chloroplast.qza --p-metric shannon --o-alpha-diversity ALPHA-shannon-vector.qza
```

Next we will generate the visual and statistical output for each result:
:::success
qiime diversity alpha-group-significance \
&nbsp;   --i-alpha-diversity ALPHA-shannon-vector.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --o-visualization ALPHA-shannon-group-significance.qzv
:::
```
qiime diversity alpha-group-significance --i-alpha-diversity ALPHA-shannon-vector.qza --m-metadata-file sample-metadata.tsv --o-visualization ALPHA-shannon-group-significance.qzv 
```
:::info

Does the active (RNA) and present (DNA) microbial community differ in these samples?

![](https://i.imgur.com/Tej3LTA.png)

:::


Now. Rerun these commands with at least one different metric. e.g. `simpson` or `chao1` in place of `shannon.`  Note that `chao1` returns the 95% confidence interval around the Chao1 estimate.  




## Beta Diversity Analyses
 
You've probably already noticed that the number of reads is quite different across samples.  Analyses of beta diversity behave very oddly if there are large differences in the number of reads across the compared samples.  Thus many programs, including QIIME, require equal number of reads across samples.  There are two different methods available in QIIME2 to do this: normalizing by "rarefying," or thowing away sequences from samples until all samples have the same number of reads, or by converting the counts in each sample to frequencies.  

* **Rarefy**: subsample the same number of sequence from each sample
* **Relative Frequency**: proportion of each feature out of the total feature counts for each sample


### Normalizing by Rarefaction

Based on the results of your rarefaction analysis, choose a sampling depth that each sample should be reduced to.  Samples where the sum of frequencies is less than the sampling depth will be not be included in the resulting table unless subsampling is performed with replacement.  Here we are randomly sampling 20,000 sequences per sample:

:::success
qiime feature-table rarefy \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast.qza  \
&nbsp;    --p-sampling-depth 20000 \
&nbsp; --p-with-replacement    \
&nbsp;    --o-rarefied-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza
:::

```
qiime feature-table rarefy  --i-table table-filtered-no-mitochondria-no-chloroplast.qza --p-sampling-depth 20000 --p-with-replacement --o-rarefied-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza
```

:::success
qiime feature-table summarize  \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  \
&nbsp;    --o-visualization table-filtered-no-mitochondria-no-chloroplast-rarefied.qzv  \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  --o-visualization table-filtered-no-mitochondria-no-chloroplast-rarefied.qzv --m-sample-metadata-file sample-metadata.tsv
```

**Export an ASV table**

ASV tables are useful for additional analyiss, e.g. for differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  --output-path ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied/feature-table.biom -o ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied/feature-table.tsv --to-tsv
```
### Normalizing by Relative Frequency 
 
:::success
qiime feature-table relative-frequency \
&nbsp;  --i-table table-filtered-no-mitochondria-no-chloroplast.qza \
&nbsp;  --o-relative-frequency-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza
:::
```
qiime feature-table relative-frequency --i-table table-filtered-no-mitochondria-no-chloroplast.qza --o-relative-frequency-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza

```
:::success
qiime feature-table summarize  \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza  \
&nbsp;    --o-visualization table-filtered-no-mitochondria-no-chloroplast-frequency.qzv  \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::
```
qiime feature-table summarize  --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza  --o-visualization table-filtered-no-mitochondria-no-chloroplast-frequency.qzv --m-sample-metadata-file sample-metadata.tsv

```
**Export an ASV table**

ASV tables are useful for additional analysis, e.g. for differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-filtered-no-mitochondria-no-chloroplast-frequency.qza  --output-path ASV-table-filtered-no-mitochondria-no-chloroplast-frequency
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-filtered-no-mitochondria-no-chloroplast-frequency/feature-table.biom -o ASV-table-filtered-no-mitochondria-no-chloroplast-frequency/feature-table.tsv --to-tsv
```

## Calculating non-phylogenetic metrics

Here we are going to calculate the beta diversity using Normalization by Relative Frequency (we will calculte some beta diversity using rarified data later on)
As with alpha diversity, there are many ways to express the beta diversity of two samples based on the presence/absence and/or relative abundances of taxa (OTUs, ASVs). The [Jaccard Index](https://en.wikipedia.org/wiki/Jaccard_index) measures similarity based on presence/absence; [Bray-Curtis](https://en.wikipedia.org/wiki/Bray%E2%80%93Curtis_dissimilarity) is a common method that combines presence/absence with relative abundance. For a full list of available metrics:
```
qiime diversity beta --help
```
:::info
[More info on alpha and beta diversity metrics](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282)
:::

We will use the 'table-filtered-no-mitochondria-no-chloroplast-**frequency**.qza' as input to beta diversity analyses, which are set up in the same manner as the alpha diversity analyses above.  Using the frequency normalized table and Jaccard:
:::success
qiime diversity beta \
&nbsp;   --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza \
&nbsp;   --p-metric jaccard  \
&nbsp;   --o-distance-matrix BETA-frequency-jaccard-distance-matrix.qza
:::
```
qiime diversity beta --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza --p-metric jaccard --o-distance-matrix BETA-frequency-jaccard-distance-matrix.qza
```
This will assess significance of all pairwise combinations and across all groups using permanova:
:::success
qiime diversity beta-group-significance \
&nbsp;   --i-distance-matrix BETA-frequency-jaccard_distance_matrix.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --m-metadata-column [column of data that you want to evaluate]\
&nbsp;   --p-method permanova  \
&nbsp;   --p-pairwise \
&nbsp;   --o-visualization BETA-frequency-jaccard-group-significance-[remember to edit name to prevent overwriting].qzv
:::
```
qiime diversity beta-group-significance --i-distance-matrix BETA-frequency-jaccard-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-frequency-jaccard-group-significance-Sulfide_Treatment_nucleic_acid_time_point.qzv
```
:::info
Can you explain these results?

![](https://i.imgur.com/B2uhxIj.png)
:::



Now, run the analysis for testing whether the present (DNA) and active (RNA) microbial communities are identical, using Jaccard and at least another different metric. You can change the metric by changing the `p-metric` parameter.


##  Calculating phylogenetic metrics (Unifrac)

### Create a tree

[Uni frac](https://en.wikipedia.org/wiki/UniFrac) UniFrac is a distance metric used for comparing biological communities. It differs from beta metrics such as Bray-Curtis  in that it incorporates phylogenetic distances between observed organisms in the computation. Both weighted (quantitative) and unweighted (qualitative) variants of UniFrac are widely used in microbial ecology, where the former accounts for abundance of observed organisms, while the latter only considers their presence or absence. The method was devised by Catherine Lozupone, when she was working under Rob Knight of the University of Colorado at Boulder in 2005.

First we group the sequences using a phylogenetic tree. Trees can be "rooted" or unrooted." Rooted trees contain a root at the base of the tree, which represents the common ancestor to all of the branches, while unrooted trees display relationships among taxa without assumptions about common ancestry.  Most phylogenetic methods produce unrooted trees, but these are often rooted at the midpoint for convenience.
![](https://i.imgur.com/qidd35j.png)

There are many methods to build these trees-- each starts by aligning sequences so that homologous positions can be compared, then using an evolutionary model to infer how the sequences are related.  We will use the `phylogeny` tool in QIIME to align sequences using the [MAFFT](https://mafft.cbrc.jp/alignment/software/) algorithm and build rooted and unrooted trees using the approximate likelihood method of [fasttree](http://www.microbesonline.org/fasttree/): 
:::success
qiime phylogeny align-to-tree-mafft-fasttree \
&nbsp;   --i-sequences rep-seqs.qza  \
&nbsp;   --o-alignment aligned-rep-seqs.qza  \
&nbsp;   --o-masked-alignment masked-aligned-rep-seqs.qza \
&nbsp;   --o-tree unrooted-tree.qza  \
&nbsp;   --o-rooted-tree rooted-tree.qza 
&nbsp;   --p-n-threads 4
:::
```
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences rep-seqs.qza --o-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza  --o-rooted-tree rooted-tree.qza --p-n-threads 4
```


You can visualize the tree in the interactive tree of live website (Itol)

Upload your tree.qza here and drag and drop into the tree taxonomy.qza and table.qza. 
https://itol.embl.de/upload.cgi

![](https://i.imgur.com/ZXM2iV2.png)



### Weighted and unweighted unifrac
Now that we have a phylogeny, we can use Unifrac metrics. These metrics take in consideration the phylogenetic relationships in the community. 


This example is for unweighted Unifrac:
:::success
qiime diversity beta-phylogenetic \
&nbsp;   --i-table filtered-table-no-mitochondria-no-chloroplast-frequency.qza \
&nbsp;   --i-phylogeny rooted-tree.qza \
&nbsp;   --p-metric unweighted_unifrac \
&nbsp;   --o-distance-matrix BETA-frequency-unweighted-unifrac-distance-matrix.qza
:::

```
qiime diversity beta-phylogenetic --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza --i-phylogeny rooted-tree.qza --p-metric unweighted_unifrac --o-distance-matrix BETA-frequency-unweighted-unifrac-distance-matrix.qza
```


This will assess significance of all pairwise combinations and across all groups using permanova:
:::success
qiime diversity beta-group-significance \
&nbsp;   --i-distance-matrix BETA-frequency-unweighted-unifrac-distance-matrix.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --m-metadata-column [column of data that you want to evaluate]\
&nbsp;   --p-method permanova  \
&nbsp;   --p-pairwise \
&nbsp;   --o-visualization BETA-frequency-unweighted-unifrac-distance-matrix-[column of data that you want to evaluate].qzv

:::
```
qiime diversity beta-group-significance --i-distance-matrix BETA-frequency-unweighted-unifrac-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-frequency-unweighted-unifrac-distance-matrix-Sulfide_Treatment_nucleic_acid_time_point.qzv

```
:::
Now, run these statistical analysis using the weighted unifrac metric. 
Why? Because Weighted UniFrac also accounts for the relative abundance of lineages between communities

:::info
Could you describe and explain the differences between the beta diversity results sumarized these two graphs?

**unweigthed**
![](https://i.imgur.com/Q2WNxty.png)




**weighted**
![](https://i.imgur.com/XAhdRk7.png)


:::


## Visualizing Beta Diversity in PCo graphs (Emperor)

Principle Coordinates Analysis is a useful tool to represent multidimensional data such as distance matrices in two or three dimensions.  PCOA generates a two (or three) dimensional graph that represents the multidimensional distance between samples.  

To generate a PCOA matrix in qiime:
:::success
qiime diversity pcoa \
&nbsp;  --i-distance-matrix BETA-jaccard_distance_matrix.qza\
&nbsp;  --o-pcoa BETA-jaccard_pcoa_matrix.qza
:::
```
qiime diversity pcoa --i-distance-matrix BETA-frequency-jaccard-distance-matrix.qza --o-pcoa BETA-frequency-jaccard-pcoa-matrix.qza
```
Now we visualize the pcoa matrix as a plot:
:::success
qiime emperor plot \
&nbsp;--i-pcoa BETA-jaccard_pcoa_matrix.qza \
&nbsp;--m-metadata-file sample-metadata.tsv \
&nbsp;--o-visualization BETA-jaccard-pcoa-plot.qzv
:::
```
qiime emperor plot --i-pcoa BETA-frequency-jaccard-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-frequency-jaccard-pcoa-plot.qzv
```


![](https://i.imgur.com/Ik2eua9.png)

1. Each dot = a 2D representation of one sample and the entire microbial community!
2. Use the drop-down menu to analyze with different metadata variables 

Change the column to select different metadata variables and look at the results… what trends do you see in the data?

**Now do the same for the unifrac distances**

```
qiime diversity pcoa --i-distance-matrix BETA-frequency-unweighted-unifrac-distance-matrix.qza --o-pcoa BETA-frequency-unweighted-unifrac-pcoa-matrix.qza

qiime emperor plot --i-pcoa BETA-frequency-unweighted-unifrac-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-frequency-unweighted-unifrac-pcoa-plot.qzv
```

:::info
Now, run the weighted-unifrac metric. 
Can you explain these graphs?
(unweigthed left, weighted right)
![](https://i.imgur.com/2Pl60w3.png)
:::

## Using core metrics diversity statistics in QIIME2 (rarefied data)
There is a build in option to generate multiple visualizations at once. It will use the normalization by rarefied we learned before. Now that we have run some stats, we can take advantage of this option to run multiple ones in a single command. 

:::success
qiime diversity core-metrics-phylogenetic \
&nbsp;  --i-phylogeny rooted-tree.qza \
&nbsp;  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza \
&nbsp;  --p-sampling-depth 20000 \
&nbsp;  --m-metadata-file sample-metadata.tsv \
&nbsp;  --output-dir core-metrics-results
:::
Note: use the same --p sampling depth as you did before in the rarify command!!
```
qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza   --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --p-sampling-depth 20000   --m-metadata-file sample-metadata.tsv --output-dir core-metrics-results

```
Now run the alpha-group-significance for each of the alpha diversity meassures. 

```
qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/observed_features_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/observed-features-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/evenness_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/evenness-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/faith_pd-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/shannon_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/shannon-group-significance.qzv

```
And now, for the beta diversity. Remember to selected the right  --m-metadata-column to run statistical tests with different variables, these variables should rlefect your hypothesis: Does microbial community structure differ significantly among different samples according to X (site, vegetation…) ? 

REMEMBER CHANGING [column] to the variable you want to test!

```
qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/unweighted-unifrac-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/weighted-unifrac-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/bray_curtis_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/bray_curtis-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/jaccard_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/jaccard-[column]-significance.qzv  --p-pairwise

```

## Comparing rarified versus frequency based normalizations

Here the graphs sumarizing the Beta diversity results using the metric unweighted unifrac and a frequency-based normalization (left) and normalization by rarefaction. 
![](https://i.imgur.com/CKQ77ZO.png)

While overall, the tendencies are the same (e.g. look to the percentage of variability explaied by each axis), there are small changes in the results (e.g. look to the position of different samples). 

:::info 
Can you compare the results of these two types of normalization using the Jaccard metric?
:::


# Visualizing your results

## How to present a long p-value list of results?
Quite often, you might have a long list of results when testing for beta diversity signinficance. One way of sumarizing pvalues is by generating a color-coded matrix.

[Example file here](https://docs.google.com/spreadsheets/d/1MrPo37VjbgAknfJ54cXetKfgiRUcsjFVC172oq6WHMs/edit?usp=sharing)

![](https://i.imgur.com/Gl4p3kb.png)



## Identifying shared and exclusive taxa (Venn diagrams). 
You can use a Venn diagram to identify the presence of shared taxa in all your treatments of those that are only found in one condition. 

You probably want to reduce a bit the degree of detail of this analysis and just focus in the species-level. For that, use the collapsed taxonomy table to compare taxomony among different types of samples using Venn diagrams. If you want to compare treatments, you will have to identify which sample belongs to each treatment and add the counts of each sample. For an example see [here ](https://docs.google.com/spreadsheets/d/1PX9prHu8zoX-chfppxebgJqi5YzJ6U2m/edit?usp=sharing&ouid=117013565017338924493&rtpof=true&sd=true)

Take all of the taxa or ASV names of the samples that you want to analyze, and put them into any of this Venn Diagram tools online:

https://bioinfogp.cnb.csic.es/tools/venny/
http://bioinformatics.psb.ugent.be/webtools/Venn/ 
http://www.interactivenn.net/

![](https://i.imgur.com/XUb6OKY.png)



## Bubble plots
You might want to sumarize the fate during the experiments of some bacterial groups of interest. As for many of the analysis you could use the ASV level or the collapsed taxonomy. Because we want to compare the abundance on each taxa, and this is a quantitative meassure, you need to use a normalized table. 
We can work with the `table-filtered-no-mitochondria-no-chloroplast-rarefied.qza` file we generated while calculating beta diversity. 
If you are interest in the ASV level, you can use the feature table within the folder `ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied`

You wnat to focus on a higher taxonomical rank, so we can easily generate a  collapsed taxonomy as we did before. 


```
qiime taxa collapse  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --i-taxonomy taxonomy.qza   --o-collapsed-table table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3.qza --p-level 3
```
You can export the table in to excel for additional analysis.  
```
qiime tools export   --input-path table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3.qza  --output-path table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3
```

Convert the exported table from biom format to tsv
```
biom convert -i table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3/feature-table.biom -o table-collapsed-filtered-no-mitochondria-no-chloroplast-rarefied-level3/feature-table.tsv --to-tsv
```
You can sort the taxa an select those that are of interest. There is a few transformation of the data you have to do. You can follow the instructions in this [example file](https://docs.google.com/spreadsheets/d/1-NiKGtMQ4TiaIEWu3uTKF6NbbfQILORhIedOdeI7ZbY/edit?usp=sharing). **Remember to modify the headers of the taxa (simplify to a single word)**
Once you have the final set of data, copy into a text file called `bubbleplot_comple.txt`

We are going to plot the abundance results for the Desulfobacterota. 

Open R, set the working directory and run this R.script. 

here is the [R script]

```
library(ggplot2)
library(reshape2)
library(viridis)
library(viridisLite)

pc = read.table("bubbleplot_comple.txt", header = TRUE)
pcm = melt(pc, id = c("ID", "GROUP")) ## GROUP is the column where you put type of sample

pcm$ID <- factor(pcm$ID,levels=unique(pcm$ID))
xx = ggplot(pcm, aes(x = ID, y = variable)) + 
  geom_point(aes(size = value, fill = GROUP), alpha = 0.75, shape = 21) + 
  scale_size_continuous(limits = c(1,10000), range = c(1,10), breaks = c(20,50,70,100,1000, 2000, 5000,10000)) + 
  labs( x= "", y = "", size = "Abundance (Rarified)", fill = "GROUP")  + 
  theme(legend.key=element_blank(), 
        axis.text.x = element_text(colour = "black", size = 7, face = "bold", angle = 90, vjust = 0.3, hjust = 1), 
        axis.text.y = element_text(colour = "black", face = "bold", size = 9), 
        legend.text = element_text(size = 10, face ="bold", colour ="black"), 
        legend.title = element_text(size = 11, face = "bold"), panel.background = element_blank(), 
        panel.border = element_rect(colour = "black", fill = NA, size = 1.2), 
        legend.position = "right", panel.grid.major.y = element_line(colour = "grey95")) +  
        scale_fill_viridis_d() +
        scale_y_discrete(limits = rev(levels(pcm$variable))) 

ggsave(file="bubble_comple.pdf", width = 9, height = 4.5, dpi = 150, units = "in", device = "pdf")


```

![](https://i.imgur.com/psemrXk.png)



# Part II. In depth analysis

![](https://i.imgur.com/sxiUUl8.png)


## Differential abundance analysis using balances in gneiss.

https://docs.qiime2.org/2021.8/tutorials/gneiss/

The main problem that we will focus on is how to identify differentially abundant taxa in a compositionally coherent way.

Compositionality refers to the issue of dealing with proportions. To account for differences in sequencing depth, microbial abundances are typically interpreted as proportions (e.g. relative abundance). Because of this, it becomes challenging to infer exactly which microbes are changing – since proportions add to one, the change of a single microbe will also change the proportions of the remaining microbes.

Consider the following example:
![](https://i.imgur.com/ThH2TIn.png)
On the left, we have the true abundances of ten species, and the first species doubles between Time point 1 and Time point 2. When we normalize these to proportions, it appears as if all of the species have changed between the two time points. Looking at proportions alone, we would never realize this problem, and we actually cannot exactly determine which species are changing based on proportions alone.

While we cannot exactly solve the problem of identifying differentially abundant species, we can relax this problem and ask which partitions of microbes are changing. In the case above, if we compute the ratio between the first species and the second species, that ratio will be 1:1 at Time point 1, and 2:1 at Time point 2 for both the original abundances and the proportions. This is the type of question that balances try to solve. Rather than focusing on individual taxa, we can focus on the ratio between taxa (or groups of taxa), since these ratios are consist between the true abundances and the observed proportions of the species observed. We typically log transform these ratios for improved visualization (‘log ratios’). The concept of calculating balances (or ratios) for multiple species can be extended to trees as shown in the following example.

![](https://i.imgur.com/RT3WU0l.png)

On the left, we define a tree, where each of the tips corresponds to a taxon, and underneath are the proportions of each taxon in the first sample. The internal nodes (i.e. balances) define the log ratio between the taxa underneath. On the right is the same tree, and underneath are the proportions of each taxa in a different sample. Only one of the taxa abundances changes. As we have observed before, the proportions of all of the taxa will change, but looking at the balances, only the balances containing the purple taxa will change. In this case, balance b3 won’t change, since it only considers the ratio between the red and taxa. By looking at balances instead proportions, we can eliminate some of the variance by restricting observations to only focus on the taxa within a given balance.

The outstanding question here is, how do we construct a balance tree to control for the variation, and identify interesting differentially abundant partitions of taxa? In gneiss, there are three main ways that this can be done:

1. Correlation clustering. 
2.  Gradient clustering. 
3.  Phylogenetic analysis. 
Once we have a tree, we can calculate balances

### Correlation clustering {categorical variable}

:::warning 
If we don’t have relevant prior information about how to cluster together organisms, we can group together organisms based on how often they co-occur with each other. This is available in the correlation-clustering command and creates tree input for ilr-hierarchical.
To simplify the analysis, and for it to run faster, we will use the collapsed taxonomy table.

:::
```
qiime gneiss correlation-clustering --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza --o-clustering GNEISS-correlation-hierarchy.qza
```
We will select the column of the metadata that better represents of hypothesis and run the analysis in regular and log scale. *Remember to change [column] (in this case the nucleid_acid) for the column name of interest*

```
qiime gneiss dendrogram-heatmap --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --i-tree GNEISS-correlation-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method clr  --m-metadata-column nucleic_acid  --p-color-map seismic  --o-visualization GNEISS-clustering-nucleic_acid-heatmap.qzv

qiime gneiss dendrogram-heatmap --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --i-tree GNEISS-correlation-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method log  --m-metadata-column nucleic_acid  --p-color-map viridis  --o-visualization GNEISS-clustering-log-nucleic_acid-heatmap.qzv

```
![](https://i.imgur.com/nhQi6XY.png)



### Gradient clustering {numerical variable}

:::warning 
Use a metadata category (numerical, e.g. days) to cluster taxa found in similar sample types. For example, if we want to evaluate if pH is a driving factor, we can cluster according to the pH that the taxa are observed in, and observe whether the ratios of low-pH organisms to high-pH organisms change as the pH changes. This is available in the gradient-clustering command and creates tree input for ilr-hierarchical.

e.g. time_point
:::
```
qiime gneiss gradient-clustering --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza --m-gradient-file sample-metadata.tsv  --m-gradient-column [numericalcolumn] --o-clustering GNEISS-gradient-hierarchy.qza
```
We will select the column of the metadata that better represents of hypothesis and run the analysis in regular and log scale. *remember to change [column- in this case the run the categorical version of that numerical version,  column can be time_point2] for the column name of interest*

```
qiime gneiss dendrogram-heatmap --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --i-tree GNEISS-gradient-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method clr  --m-metadata-column [column]  --p-color-map seismic  --o-visualization GNEISS-clustering-[column]-heatmap.qzv

qiime gneiss dendrogram-heatmap --i-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --i-tree GNEISS-gradient-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method log  --m-metadata-column [column]  --p-color-map viridis  --o-visualization GNEISS-clustering-log-[column]-heatmap.qzv
```


### Phylogenetic analysis.

:::warning
A phylogenetic tree (e.g. rooted-tree.qza) created outside of gneiss can also be used. In this case you can use your phylogenetic tree as input for ilr-phylogenetic.
here you **can not** use the collapsed taxonomy table. 
:::

```
qiime gneiss ilr-phylogenetic --i-table table-filtered-no-mitochondria-no-chloroplast.qza --i-tree rooted-tree.qza --o-balances balances --o-hierarchy GNEISS-phylogenetic-hierarchy.qza

```

We use the rooted-tree.qza from previous analyisis to generate the hierarchy file  and we will select the column of the metadata that better represents of hypothesis and run the analysis in regular and log scale. *remember to chage [column- in this case the nucleid_acid] for the column name of interst*

```
qiime gneiss dendrogram-heatmap --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --i-tree GNEISS-phylogenetic-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method clr  --m-metadata-column nucleic_acid  --p-color-map seismic  --o-visualization GNEISS-phylogenetic-nucleic_acid-heatmap.qzv

qiime gneiss dendrogram-heatmap --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --i-tree GNEISS-phylogenetic-hierarchy.qza  --m-metadata-file sample-metadata.tsv --p-method log  --m-metadata-column nucleic_acid  --p-color-map viridis  --o-visualization GNEISS-phylogenetic-log-nucleic_acid-heatmap.qzv
```
![](https://i.imgur.com/xQAlWFY.png)



:::warning
Could you ran the correlation and gradient differential abundance analysis with the non-collapsed table? and with other variables?
:::

## Differential abundance ANCOM for identication of taxa of interest
[tutorial](
https://docs.qiime2.org/2021.8/tutorials/pd-mice/?highlight=gneiss#differential-abundance-with-ancom)


Many microbiome investigators are interested in testing whether individual ASVs or taxa are more or less abundant in different sample groups. This is known as differential abundance. Microbiome data present several challenges for performing differential abundance using convential methods. Microbiome abundance data are inherently sparse (have a lot of zeros) and compositional (everything adds up to 1). Because of this, traditional statistical methods that you may be familiar with, such as ANOVA or t-tests, are not appropriate for performing differential abundance tests of microbiome data and lead to a high false-positive rate. ANCOM is a compositionally aware alternative that allows to test for differentially abundant features.

Before we begin, we will filter out low abundance/low prevalence ASVs. Filtering can provide better resolution and limit false discovery rate (FDR) penalty on features that are too far below the noise threshhold to be applicable to a statistical test. A feature that shows up with 10 counts could be a real feature that is present only in that sample; a feature that’s present in several samples but only got amplified and sequenced in one sample because PCR is a somewhat stochastic process; or it may be noise. It’s not possible to tell, so feature-based analysis may be better after filtering low abundance features. However, filtering also shifts the composition of a sample, further disrupting the relationship. Here, the filtering is performed as a trade off between the model, computational efficiency, and statistical practicality.


###ASV analysis
```
qiime feature-table filter-features --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --p-min-frequency 50  --p-min-samples 2 --o-filtered-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund.qza
  ```

ANCOM fundamentally operates on a FeatureTable[Frequency], which contains the frequencies of features in each sample. However, ANCOM cannot tolerate zeros because compositional methods typically use a log-transform or a ratio and you can’t take the log or divide by zeros. To remove the zeros from our table, we can add a pseudocount to the FeatureTable[Frequency] Artifact, creating a FeatureTable[Composition] in its place.

```
qiime composition add-pseudocount --i-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund.qza  --o-composition-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund-comp.qza
```

Let’s use ANCOM to check whether there is a difference in the mice based on their donor and then by their genetic background. The test will calculate the number of ratios between pairs of ASVs that are significantly different with FDR-corrected p < 0.05. *Remember to choose the column of interest*.

```
qiime composition ancom --i-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund-comp.qza --m-metadata-file sample-metadata.tsv --m-metadata-column nucleic_acid  --o-visualization ANCOM-table-filtered-no-mitochondria-no-chloroplast-nucleic_acid.qzv
```
![](https://i.imgur.com/V63pMxx.png)

:::info

Now, run the ANCOM with the collapsed taxonomy.
CAn you explain the differences between the graphs?
What are the advantages of the ASV analysis?
and of the taxonomy-based grouping?

![](https://i.imgur.com/eEhQxzn.png)


:::

## Longitudinal analysis

These analysis require at least two time points for comparison. Full tutorial [here](https://docs.qiime2.org/2019.10/tutorials/longitudinal/).

![](https://i.imgur.com/Ih3c5Px.png)



### Pairwise difference comparisons
Pairwise difference tests determine whether the value of a specific metric changed significantly between pairs of paired samples (e.g., pre- and post-treatment).

This visualizer currently supports comparison of feature abundance (e.g., microbial sequence variants or taxa) in a feature table, or of metadata values in a sample metadata file. Alpha diversity values (e.g., observed sequence variants) and beta diversity values (e.g., principal coordinates) are useful metrics for comparison with these tests, and should be contained in one of the metadata files given as input. In the example below, we will test whether alpha diversity (Shannon diversity index) changed significantly between two different time points in the ECAM data according to delivery mode


`--p-state-column` needs to be numerical. (usually time)
`--p-metric` needs to be numerical.
`--p-individual-id-column` needs to be categorial (this is what you are testing, the treatment). THis colunm identifing the samples that belong to the same individual through time
`--p-baseline` first time point. (lowest value in `--p-state-column` )

```
qiime longitudinal pairwise-differences   --m-metadata-file sample-metadata.tsv   --m-metadata-file ALPHA-shannon-vector.qza   --p-metric shannon_entropy   --p-group-column Sulfide_Treatment   --p-state-column time_point   --p-state-1 1   --p-state-2 18   --p-individual-id-column nucleic_acid   --p-replicate-handling random   --o-visualization pairwise-differences-nucleic_acid.qzv
```

![](https://i.imgur.com/YxmbiCO.png)

### Pairwise distances comparisons
The pairwise-distances visualizer also assesses changes between paired samples from two different “states”, but instead of taking a metadata column or artifact as input, it operates on a distance matrix. You can find discance matrices in the core-metric-results folder. 



```
qiime longitudinal pairwise-distances  --m-metadata-file sample-metadata.tsv   --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza      --p-group-column Sulfide_Treatment_nucleic_acid --p-state-column time_point   --p-state-1 1   --p-state-2 18   --p-individual-id-column nucleic_acid   --p-replicate-handling random   --o-visualization pairwise-distances-unweighted_unifrac_distance_matrix.qzv
```

### Volatility analysis
The volatility visualizer generates interactive line plots that allow us to assess how volatile a dependent variable is over a continuous, independent variable (e.g., time) in one or more groups. Multiple metadata files (including alpha and beta diversity artifacts) and FeatureTable[RelativeFrequency] tables can be used as input, and in the interactive visualization we can select different dependent variables to plot on the y-axis.

Here we examine how variance in Shannon diversity and other metadata changes across time (set with the state-column parameter) in the ECAM cohort, both in groups of samples (interactively selected as described below) and in individual subjects (set with the individual-id-column parameter).
`--p-individual-id-column colunm identifing the samples that belong to the same individual through time`

We can look to changes in the beta diversity indexes
```
qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file BETA-frequency-unweighted-unifrac-pcoa-matrix.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point --p-default-metric 'Axis 2'  --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_unweight_frequency_pc_vol.qzv

```

![](https://i.imgur.com/lg2xiwc.png)

or to changes, for example in the number of taxa observed. 

```
 qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file ./core-metrics-results/observed_features_vector.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point   --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_observed_feature_pc_vol.qzv
```

:::success

*how to run weighted-unifrac* 

```

qiime diversity beta-phylogenetic --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza --i-phylogeny rooted-tree.qza --p-metric weighted_unifrac --o-distance-matrix BETA-frequency-weighted-unifrac-distance-matrix.qza


qiime diversity beta-group-significance --i-distance-matrix BETA-frequency-weighted-unifrac-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-frequency-weighted-unifrac-distance-matrix-Sulfide_Treatment_nucleic_acid_time_point.qzv

qiime diversity pcoa --i-distance-matrix BETA-frequency-weighted-unifrac-distance-matrix.qza --o-pcoa BETA-frequency-weighted-unifrac-pcoa-matrix.qza

qiime emperor plot --i-pcoa BETA-frequency-weighted-unifrac-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-frequency-weighted-unifrac-pcoa-plot.qzv

qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file BETA-frequency-weighted-unifrac-pcoa-matrix.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point --p-default-metric 'Axis 2'  --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_unweight_frequency_pc_vol.qzv

```



:::


![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 