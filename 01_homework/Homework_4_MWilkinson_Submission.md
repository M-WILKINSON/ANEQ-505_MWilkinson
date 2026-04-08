
------------------------------------------------------------------
Due: 04/09/2026
--------------------------------------------------
#### Cow Body Site - making figures in R
**Set up the cow R analysis file structure**
1. Make a cow_r directory on your local computer, and inside the cow_r directory,
make the following directories
cow_r
├── 01_notes
├── 02_data
├── 03_metadata
├── 04_code
└── 05_figures

2. Inside the ***02_data directory***, make the following directories 
(NOTE: I updated the designated folder from code to data since these are the input data files and per mirroring last Friday's decomp tutorial: 

"The **`01_notes`** directory is where you can store any notes related to your project.  
The **`02_data`** directory should contain raw data (for example, sequencing data, or qiime2 outputs qzvs).  
The **`03_metadata`** directory will store all metadata associated with the project.  
The **`04_code`** directory is where your R scripts will be kept.   
The **`05_figures`** directory will store figures generated from your R analyses.)"

02_data
├── alpha_div
├── beta_div
├── taxonomy

3. Download the cow_metadata.txt, shannon.tsv, unweighted_unifrac.txt,
tabulated_results.tsv, and cow_HW4_r.Rmd files from Canvas and put them in the
correct directories.
**What directory should the cow_HW4_r.Rmd file go in?
The rmd file is stored in 04_code because it contains the code script commands we will run in R. 

#### Statistical analysis and figure generation in R
Now that we have set up the correct file structure and put our files in the
correct directories, we can start our cow R analysis.

4. Open the cow_HW4_r.Rmd file and start working through the analysis.

**Read in metadata~**
4. Fill in the file path you used in the R Markdown to load the metadata.
```
metadata <- read_tsv("C:/Users/Melan/Obsidian/cow_r/03_metadata/cow_metadata.txt")
```

**Read in alpha diversity data**
6. Fill in the file path you used in the R Markdown to load the shannon data
```
shannon <- read_tsv("C:/Users/Melan/Obsidian/cow_r/02_data/alpha_div/shannon.tsv")
```

**Read in beta diversity data**
7. Fill in the file path you used in the R Markdown to load the unweighted unifrac
data
```
uw_unifrac <- read_tsv("uw_unifrac <- read_tsv("C:/Users/Melan/Obsidian/cow_r/02_data/beta_div/unweighted_unifrac.txt")
```

**Load in tabulated results**
8. Fill in the file path you used in the R Markdown to load the
tabulated_results.tsv
```
tabulated_results <- read_tsv("C:/Users/Melan/Obsidian/cow_r/02_data/taxonomy/tabulated_results.tsv")
```

#### Cow Body Site - ANCOM-BC2 in Qiime2
9. **Start an interactive session and activate Qiime2**
```
ainteractive --ntasks=4 --time=04:00:00
```

10.  **ANCOMBC2 is only available in the 2026 versions of qiime2, so we need to
activate the latest version. Make sure to activate qiime2026**
```
module purge
module load qiime2/2026.1_amplicon
```
(When running commands using qiime2/2026.1_amplicon you might get this warning:
*/curc/sw/install/bio/qiime2/2026.1/2026.1_amplicon_env/lib/python3.10/site-
packages/unifrac/__init__.py:9: UserWarning: pkg_resources is deprecated as an API.
See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources
package is slated for removal as early as 2025-11-30. Refrain from using this
package or pin to Setuptools<81. import pkg_resources*. This is just saying that
one of the qiime2 packages needs to be updated it won't affect the qiime2 outputs.)
11. **Filter controls out of our table
```
# Get matadata with no controls
cp /pl/active/courses/2025_summer/CSU_2025/cow_hw/cow_metadata_nocontrols.txt .
qiime feature-table filter-samples \--i-table
../dada2/table_nomitochloro_gg2_filtered300.qza \--m-metadata-file
cow_metadata_nocontrols.txt \--o-filtered-table
table_nomitochlorocontrols_gg2_filtered300.qza
```
**Filter Samples**
12. Navigate into the cow tutorial and make a new ancombc2 directory for the ANCOM-BC2 analysis and navigate into the ancombc2 directory
```
mkdir ancombc2  
  
cd ancombc2
```

12. Choose the min frequency for sample filtering:
```
qiime feature-table filter-samples \--i-table
table_nomitochlorocontrols_gg2_filtered300.qza \--p-min-frequency 5000
\--o-filtered-table table_5k.qza
```

14. **Filter out low abundance and low prevalence ASVs**
```
qiime feature-table filter-features \  
--i-table table_5k.qza \  
--p-min-frequency 50 \  
--p-min-samples 20 \  
--o-filtered-table table_5k_abund.qza
```

**Collapse features to genus level**
15. We will collapse to the genus level to make it easier to interpret the results.
(Hint: We used 7 for species, so think about which number you would use for genus.)
```
qiime taxa collapse \  
--i-table table_5k_abund.qza \  
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \  
--p-level 6 \  
--o-collapsed-table table_5k_abund_6.qza
```

16. **Run ANCOM-BC2**
```
qiime composition ancombc2 \  
--i-table table_5k_abund_6.qza \  
--m-metadata-file ../cow_metadata_nocontrols.txt \  
--p-fixed-effects-formula body_site \  
--o-ancombc2-output ancombc2_results_bodysite_genus.qza
```

17. **Visualize the ANCOM-BC2 results**
 Generate a barplot to visualize the differentially abundant features.
```
qiime composition tabulate \  
--i-data ancombc2_results_bodysite_genus.qza \  
--o-visualization ancombc2_bodysite_genus.qzv  
  
qiime composition ancombc2-visualizer \  
--i-data ancombc2_results_bodysite_genus.qza \  
--o-visualization ancombc2_barplot_bodysite_genus.qzv
```

## Homework questions: 
1. Describe one way to get data from your qiime2 outputs into a format that can be used for R.

We can download our alpha and beta diversity metrics and the tabulated_results.tsv file from Alpine OnDemand and then unzip and load (read) them into R using the read function. 

2. Which body site appeared most distinct in the taxa bar plot, meaning it was not
similar to at least one of the other body sites? Explain why that site looks
different.

The fecal samples were most different to all the other sample types. ![[PCoA_fecal_cow_hw4.png]] In addition to analyzing the taxabarplots, we can also see in the PCoA plot that they are group separately/all on their own in the upper left-hand corner rather than overlapping with other sample group types. 


3. When generating the filtered table for ANCOM-BC2, what value did you choose for
`--p-min-frequency`? Which core metrics parameter should this match, and why do
these values need to be the same? (Report your core metrics value here:  5,000)

I used 5000 for `--p-min-frequency`. This should match the sampling depth (`--p-sampling-depth`) from the core metrics step, which was also set to 5000. These need to match so we’re working with the same set of samples across analyses. If they’re different, you could end up including samples in one step that were excluded in another, which would make the results harder to compare and potentially less reliable.

4. Why do we filter out samples with low frequency and low abundance ASVs?

We filter out low-frequency samples and low-abundance ASVs in order to clean up the data. Samples with very low read counts aren’t very reliable, and really rare ASVs are more likely to be noise or sequencing errors rather than true biological signals. Removing them helps reduce noise and makes the results from analyses possible to interpret.

5. What was the most enriched genus in skin compared to fecal, and what was the most depleted genus in skin compared to fecal (make sure adjusted p is set to less
than 0.05)? 




## Extra credit~ generate a classification model to see how well we can predict cow body site
```
cd /scratch/alpine/c832787271@colostate.edu/cow

mkdir ml
cd ml

# Remove controls from the feature table  
qiime feature-table filter-samples \  
--i-table ../core_metrics_results/rarefied_table.qza \  
--m-metadata-file ../metadata/cow_metadata.txt \  
--p-where "[body_site] != 'control'" \  
--o-filtered-table rarefied_table_no_controls.qza  
  
# Collapse features to taxonomic level 7 (species)  
qiime taxa collapse \  
--i-table rarefied_table_no_controls.qza \  
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \  
--p-level 7 \  
--o-collapsed-table rarefied_table_no_controls_L7.qza
```

```
qiime sample-classifier classify-samples --i-table rarefied_table_no_controls_L7.qza --m-metadata-file ../cow_metadata_nocontrols.txt --m-metadata-column body_site --p-random-state 123 --output-dir sample_classifier_results_bodysite
```



### **Questions:**
1. Why might removing controls be important before downstream analysis?

Removing controls is important because they don’t represent real samples. Including them could skew results or reduce the accuracy of analyses, so we remove them to focus only on true biological variation.

2. What 2 features that are high in fecal samples?

3. What are 2 features that are low in nasal?

4. What is the accuracy of your model, and if the accuracy of the classifier is high, what does that suggest about the microbial compositions of each site?

The model achieves 88% overall accuracy, far above the 35% baseline, with classification for fecal, nasal, oral, and skin samples, while udder samples show some overlap with skin. This indicates that most body sites have distinct microbial communities, though udder and skin share some similarities.


Model Accuracy Table (accuracy_results.qzv):
|fecal|nasal|oral|skin|udder|Overall Accuracy|
|---|---|---|---|---|---|
|fecal|1.0|0.0|0.0|0.0|0.0||
|nasal|0.0|1.0|0.0|0.0|0.0||
|oral|0.0|0.0|1.0|0.0|0.0||
|skin|0.0|0.0|0.0|1.0|0.0||
|udder|0.0|0.0|0.0|0.666667|0.333333||
|Overall Accuracy||||||0.882353|
|Baseline Accuracy||||||0.352941|
|Accuracy Ratio||||||2.5|