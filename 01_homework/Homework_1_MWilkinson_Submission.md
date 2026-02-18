2026-02-16

Step 1. Logged into Alpine via OnDemand and created a new directory called "cow" using the new directory button. 

Step 2. Clicked on "open in terminal", so it was already in the correct cow directory but if I was in another instance/terminal I would use the following command to change directories:
```
cd /scratch/alpine/c832787271@colostate.edu/cow
```

Step 3. Created my sub-directories
```
mkdir slurm
mkdir taxonomy
mkdir tree
mkdir taxaplots
mkdir dada2
mkdir demux
mkdir metadata
mkdir core_metrics

ls
```

Step 4. Upload metadata files (cow_barcodes.txt, cow_metadata.txt) in OnDemand using the upload files button after navigating to the metadata sub folder. 

Step 5 & 6. Opened a new terminal in the cow directory and copied the raw sequencing files from shared public folder. 
```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```

Step 5. Launch interactive session and load qiime2
```
ainteractive --ntasks=6 --time=02:00:00
module purge
module load qiime2/2024.10_amplicon
```

Step 6. Import the sequences/reads into a Qiime2-readable format (.qza).
```
qiime tools import \--type EMPPairedEndSequences \--input-path raw_reads \--output-path cow_reads.qza
```

Step 7. Demultiplex the reads by submitting a job
a. Navigated to the slurm directory in Alpine in OnDemand and created a new file called demux.sh, then ran demultiplexing command:
```
#!/bin/bash
#SBATCH --job-name=demux
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=melanie.wilkinson@colostate.edu

#What needs to go here in order to “turn on” qiime2? Hint: we do these 2 commands every time we activate qiime2!
module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/c832787271@colostate.edu/cow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \--m-barcodes-file ../metadata/cow_barcodes.txt \--m-barcodes-column barcode \--p-rev-comp-mapping-barcodes \--p-rev-comp-barcodes \--i-seqs ../cow_reads.qza \--o-per-sample-sequences demux_cow.qza \--o-error-correction-details cow_demux_error.qza

#visualize the read quality
qiime demux summarize \--i-data demux_cow.qza \--o-visualization demux_cow.qzv
```

b. Run the script in the slurm directory as a job. 
``` 
dos2unix demux.sh
sbatch demux.sh
```

Step 8. I reviewed the interactive quality plot in qiime2 view and for the forward reads quality scores looked good (~30), and for reverse only 1 fell significantly below 30 (13) and that was after 250, therefore, I decided to use 250 as my threshold #. 
```
cd /scratch/alpine/c832787271@colostate.edu/cow/dada2

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_cow.qza \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--o-representative-sequences cow_seqs_dada2.qza \--o-denoising-stats cow_dada2_stats.qza \--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \--m-input-file cow_dada2_stats.qza \--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \--i-table cow_table_dada2.qza \--m-sample-metadata-file ../metadata/cow_metadata.txt \--o-visualization cow_table_dada2.qzv

qiime feature-table tabulate-seqs \--i-data cow_seqs_dada2.qza \--o-visualization cow_seqs_dada2.qzv
```

Answers to questions:
Briefly **describe** the key information from each denoising output file:
 *I have not been able to successfully run the denoising job*
1. Representative Sequences
2. Denoising Stats
3. Denoised Table

**Answer the following questions:**

1. Where does the median Q-score begin to dip below Q30 for the forward reads and the reverse reads? For the forward reads, it appeared they were all equal to or greater than 30. For the reverse reads we had one dip down to 13 at position 251 (the very last read), and therefore filtered that one out. 
2. What is the mean reads per sample? 15,163.39
3. How long are the reads? 251 nts
4. What is the maximum length of all your sequences? 43,963
5. Which sample (not including extraction controls starting with EC) lost the highest % of reads? Position 251.
6. Why did you chose to trim or truncate where you did? After reviewing the interactive quality plot in qiime2 view and for the forward reads quality scores looked good (~30), and for reverse only 1 fell significantly below 30 (13) and that was after 250, therefore, I decided to use 250 as my threshold #.