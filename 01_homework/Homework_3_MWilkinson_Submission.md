
**Cow Site Data Workflow**,  Part 3

 Load interactive session
```
ainteractive --ntasks=6 --time=02:00:00
```

Load qiime2 in a terminal session after you go into the **cow** folder
```
# Insert the two commands to activate qiime2

module purge  
module load qiime2/2024.10_amplicon  
```

### Alpha Rarefaction Plot 

Choose the input sequencings depths (min and max) for generating the alpha rarefaction plot:
```

#go to the cow directory

cd /scratch/alpine/c832787271@colostate.edu/cow

qiime diversity alpha-rarefaction \--i-table dada2/cow_table_dada2_filtered300.qza \--m-metadata-file metadata/cow_metadata.txt \--o-visualization alpha_rarefaction_curves_16S.qzv \--p-min-depth 1000 \--p-max-depth 10000

```

### Run Core Metrics 

```

qiime diversity core-metrics-phylogenetic \--i-table dada2/cow_table_dada2_filtered300.qza \--i-phylogeny tree/tree_gg2.qza \--m-metadata-file metadata/cow_metadata.txt \--p-sampling-depth 1500 \--output-dir core_metrics_results

```

### Visualize alpha diversity plots

Generate a plot to visualize the observed features
```

qiime diversity alpha-group-significance \--i-alpha-diversity core_metrics_results/observed_features_vector.qza \--m-metadata-file metadata/cow_metadata.txt \--o-visualization core_metrics_results/observed_features_statistics.qzv

```

Generate a plot to visualize faith's PD 
```

## insert the entire code chunk for generating this visualization

qiime diversity alpha-group-significance \--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \--m-metadata-file metadata/cow_metadata.txt \--o-visualization core_metrics_results/faiths_pd_statistics.qzv  
  
```

## Homework questions:
1. What is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample)

The table.qzv file is the one used to visualize the sequencing depth across all of our samples. In the cow dataset, this was the [cow_table_dada2_filtered300.qzv] file. 

2. What did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why?

I pulled the table.qzv file into QIIME2 view and started with taking a look at the overview tab, specifically observing the min (3), median (9,027), and max (33,768) frequencies per sample to better understand the range. Then I looked at the interactive sample detail and feature detail tabs to understand in greater detail my feature representation across samples.  

A minimum depth of 1,000 was selected to exclude extremely low-depth samples, while a maximum depth of 10,000 was chosen because it is near the median sequencing depth and retains the majority of samples before substantial sample loss occurs.


3. Which cow body location had more observed features? Which has the lowest?

While referencing [alpha_rarefaction_curves_16S.qzv] in QIIME2 view, it appears that the fecal samples had the highest number of observed features and the nasal samples had the least. 

4. What is the main difference between Faiths PD and Shannon's alpha diversity metrics?  

Faith’s PD uses evolutionary relationships between organisms, while Shannon only uses abundance and richness.

5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information?

For alpha diversity, Faith's PD requires tree information and for beta diversity the weighted and unweighted unifrac also requires tree information. 

6. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different?

The fecal, skin, and udder samples have higher phylogenetic diversity than the controls, nasal, or oral samples.

7. Does it seem like there are any groupings in the beta diversity? What are the groupings?

8. Why do you think these samples are grouping together?

Different environments naturally select for different microbial communities. Could be based off of environmental conditions and required functions of microbes. 

9. What test can you run to determine if the groups are significantly different?

A PERMANOVA test.

10. What command would you use to run that test?
```

#insert command for running the test you suggest from question 7

qiime diversity beta-group-significance \  
--i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \  
--m-metadata-file metadata/sample-metadata.tsv \  
--m-metadata-column body-site \  
--p-method permanova \  
--o-visualization unweighted-unifrac-body-site-significance.qzv

```