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

Step 8. 