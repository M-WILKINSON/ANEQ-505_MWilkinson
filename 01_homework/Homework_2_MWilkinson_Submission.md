**Cow Site Data Workflow**, Part II. 

  
0. Load interactive session
```
ainteractive --ntasks=6 --time=02:00:00
```

1. Load qiime2 in a terminal session after you go into the taxonomy folder
```
cd /scratch/alpine/c832787271@colostate.edu/cow/taxonomy

# Insert the two commands to activate qiime2
module purge  
module load qiime2/2024.10_amplicon 
```

2. Remove long (300+ base pair) amplicons from the representative sequences file and the feature table
```
# filter out any large amplicons from the seqs and table (because they are contaminates)

cd /scratch/alpine/$USER/cow/dada2

qiime feature-table filter-seqs \--i-data cow_seqs_dada2.qza \--m-metadata-file cow_seqs_dada2.qza \--p-where 'length(sequence) < 300' \--o-filtered-data cow_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \--i-data cow_seqs_dada2_filtered300.qza \--o-visualization cow_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \--i-table cow_table_dada2.qza \--m-metadata-file cow_seqs_dada2_filtered300.qza \--o-filtered-table cow_table_dada2_filtered300.qza

qiime feature-table summarize \--i-table cow_table_dada2_filtered300.qza \--m-sample-metadata-file ../metadata/cow_metadata.txt \--o-visualization cow_table_dada2_filtered300.qzv
```

3. Classify taxonomy using GreenGenes2. First get the Greengenes2 database:
```
cd /scratch/alpine/$USER/cow/taxonomy
```

```
wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

4. Classify taxonomy using GreenGenes2 classify the ASVs (takes about 5 mins). 
```
qiime feature-classifier classify-sklearn \--i-reads ../dada2/cow_seqs_dada2_filtered300.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2_filtered.qza
```

5. Visualize the taxonomy of your ASVs:
```
qiime metadata tabulate \--m-input-file taxonomy_gg2_filtered.qza \--o-visualization taxonomy_gg2_filtered.qzv
```

6. Filter mitochondria and chloroplast out to generate a filtered feature table, keep only ASVs with a class or lower taxonomy. fill in the blank (--p-exclude) to exclude these DNA. Fill in the blank to include only class level or below classifications. 
```
qiime taxa filter-table \--i-table ../dada2/cow_table_dada2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

7. Visualize the taxa bar plot
```
qiime taxa barplot \--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--m-metadata-file ../metadata/cow_metadata.txt \--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```


**Filtered Taxa Bar Plot Questions:**

***Question 1: Attach a picture of your taxa bar plot, organized by cow sampling location (body_site) at the level 7 taxonomic level.*** 
![[homework-2-cow-level-7-taxaplot-bars.svg]]
Added the png because axis were more easily visible in this version. 
![[level 7 taxa barplot pasted image.png]]

**What general trends do you notice?**

The taxa bar plot shows a lot of variation in microbial composition across sample locations. Some samples are dominated by just a few taxa, while others, especially fecal samples, are more evenly distributed and diverse. The skin and udder samples look very similar to each other, and the same pattern is seen for oral and nasal samples. Overall, the composition differs by sample, but some taxa are shared across the groups.


***Question 2. What are the top 2 most abundant bacterial classes in the fecal samples?***

The most two relatively abundant bacteria classes among the fecal samples were c__Clostridia_258483 and c__Bacteroidia. 
  

***Question 3: What highly abundant ASV is shared between both the udder and skin samples?***

The most highly relative abundant ASV shared between the udder and skin samples was: "d__Bacteria;p__Bacillota_A_368345;c__Clostridia_258483;o__Oscillospirales;f__Oscillospiraceae_88309;g__Faecousia;s__Faecousia sp000434635" with a frequency of 43,310. This frequency was calculated by pulling the CSV from the taxa barplot into Python, filtered to the sample locations we were interested in, (udder & skin), using panda's groupby and sum functions. Alternatively, this frequency could be obtained from creating a simple pivot table in excel or google sheets however since bioinformatics data tend to be larger, I opted for the coding route. 
  

***Question 4: Which samples (still sorted by body_site) have higher alpha diversity in terms of observed features?***

The fecal, skin, and udder body site samples appeared to have the highest alpha diversity, or the samples with the highest number of unique features found in them. 
  

***Question 5: Do all samples contain archaea as well?***

No, not all *samples* contained archaea, however, each sample site showed the following frequencies for detected archaea across the total samples for that body site:

| control | 0 |
| fecal | 2,151 |
| nasal | 111 |
| oral | 139 |
| skin | 1,910 |
| udder | 887 |
| (blank) | 0 |
  

***Question 6: Why do we filter out sp004296775?***

We filtered out because it's a chloroplast in disguise, (has different nomenclature but it has been recognized as a chloroplast contaminant by the scientific community). In lab we learned that: the reason these show up in our data is due to the endosymbiotic theory, that mitochondria & chloroplasts, likely originating as bacteria, became symbionts in cytoplasm of eukaryotic cells.
  

***Question 7: What is the difference between these two flags?***

*--p-exclude mitochondria,chloroplast,sp004296775 \*

This parameter helps us filter out mitochondria and chloroplasts from our samples which are classified as contaminants. 

*--p-include c__ \*

This parameter helps us filter to only the well classified ASVs as a data cleaning/quality control step, helping to improve our signal to noise ratio in our data. 

***Question 8: Do the positive controls look the same as each other? Yes or No?***

Yes (similar, but not identical)
  

***Question 9: Do the negative/extraction controls (Samples labeled as EC), look like the positive controls? Yes or no?***

Yes (PC1.3 and PC1.4 share some high-level trends with EC1.4, EC2.3, but the others look different)
  

***Question 10: Do the negative/extraction controls (Samples labeled as EC), look like the real samples? Yes or no?***

No. They aren't nearly as diverse as our actual samples and their most relatively abundant or dominating taxa are completely different from what we are seeing in the cow samples. 
  

***Phylogenetic tree***

Create a job script to run the phylogenetic tree building. Remember you must start a new terminal session, navigate to your slurm directory, and then submit the job. You do NOT need to start any other interactive sessions.This job will take about an hour.

8. Go to OnDemand and create a new text file for your job script
```
nano tree.sh
```

```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=20:00:00 
#SBATCH --mail-type=ALL
#SBATCH --mail-user=melanie.wilkinson@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal


#Activate qiime
module purge  
module load qiime2/2024.10_amplicon 
  
#set working directory
cd /scratch/alpine/c832787271@colostate.edu/cow

#Get reference
wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza

  
#Command

qiime fragment-insertion sepp \--i-representative-sequences dada2/cow_seqs_dada2_filtered300.qza \--i-reference-database tree/2022.10.backbone.sepp-reference.qza \--o-tree tree/tree_gg2.qza \--o-placements tree/tree_placements_gg2.qza

```

9. Submit the job from the terminal
```
#submit the job

dos2unix tree.sh
sbatch tree.sh
```

*We will use this file in the next homework!

10. Once this job finishes, copy and paste what the slurm email says here:

Job ID: 24403847  
Cluster: alpine  
User/Group: c832787271@colostate.edu/c832787271pgrp@colostate.edu  
State: COMPLETED (exit code 0)  
Nodes: 1  
Cores per node: 8  
CPU Utilized: 03:33:35  
CPU Efficiency: 12.33% of 1-04:51:52 core-walltime  
Job Wall-clock time: 03:36:29  
Memory Utilized: 8.48 GB  
Memory Efficiency: 28.28% of 30.00 GB (3.75 GB/core)