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

Classify taxonomy using GreenGenes2 classify the ASVs (takes about 5 mins). ~={red}(1point)=~

```
qiime feature-classifier classify-sklearn \--i-reads ../dada2/cow_seqs_dada2_filtered300.qza \--i-classifier NAME OF CLASSIFIER HERE.qza \--o-classification taxonomy_gg2_filtered.qza
```