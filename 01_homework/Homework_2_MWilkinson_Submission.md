**Cow Site Data Workflow**, part 2

  
0. Load interactive session
```
sinteractive --reservation=aneq505 --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal
```

1. Load qiime2 in a terminal session after you go into the taxonomy folder
```
cd /scratch/alpine/c832787271@colostate.edu/decomp_tutorial/taxonomy

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

4. Classify taxonomy using GreenGenes2 classify the ASVs (takes about 5 mins). ~={red}(1point)=~
```
qiime feature-classifier classify-sklearn \--i-reads ../dada2/cow_seqs_dada2_filtered300.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2_filtered.qza
```

5. Visualize the taxonomy of your ASVs: (~={red}1point)=~
```
qiime metadata tabulate \--m-input-file taxonomy_gg2_filtered.qza \--o-visualization taxonomy_gg2_filtered.qzv
```

** i am here
6. Filter mitochondria and chloroplast out to generate a filtered feature table, keep only ASVs with a class or lower taxonomy. fill in the blank (--p-exclude) to exclude these DNA. Fill in the blank to include only class level or below classifications. ~={red}(1point)=~

```
qiime taxa filter-table \--i-table ../dada2/<YourDenoisedTable.qza> \--i-taxonomy taxonomy_gg2.qza \--p-exclude WHAT TO EXCLUDE HERE \--p-include WHAT TO INCLUDE HERE \--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

  
7. Visualize the taxa bar plot
```
qiime taxa barplot \--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--m-metadata-file ../metadata/cow_metadata.txt \--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```

