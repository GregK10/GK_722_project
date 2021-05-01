[Go back to the intial run of PERF](https://github.com/GregK10/GK_722_project/blob/main/1_Perf_and_the_reference_genome.md)

## Download and Qualtity check of raw reads

With PERF capable to identify the microsatellites within the A.fumimgatus reference genome, I wanted to conduct test PERF with a genome constructed from raw reads.
Therefore, I have obtained the raw reads of CM21, an _A. fumigatus strains_ obtained by my lab. The raw reads were obtained from NCBI and uploaded to Sharcnet. As these files are large and are not required after the run is finished, I created a scratch directory to store them. I also renamed the SSR for Cm21 file to SRR_cm21.

```{bash}
$ mkdir ~/scratch/reads ; cd ~/scratch/reads
 
$ wget https://sra-download.ncbi.nlm.nih.gov/traces/sra50/SRR/012591/SRR12894203
$ module load StdEnv/2020
$ module load gcc/9.3.0
$ module load sra-toolkit/2.10.8
$ mv SRR12894203 SRR_cm21
 
```
 
First the SSR needs to be converted into a fastq files. As paired end were, i used the ```--split-files``` flag. The newly created fastq files were placed in my scratch directory.

```{bash}

$ nano ~/bash_files/SSR_fastq.sh
______________________

#!/bin/bash
#SBATCH --account=######
#SBATCH --time=01:15:00
#SBATCH --cpus-per-task=8
#SBATCH --nodes=2
#SBATCH --mem=30G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

fasterq-dump --split-files $1
_______________________

$ ~/bash_files/SSR_fastq.sh SRR_cm21
$ ll reads

total 9859988
-rw-r----- 1 gk1995 gk1995 1449122449 Oct 26  2020 SRR_cm21
-rw-r----- 1 gk1995 gk1995 4322705158 Apr 27 21:04 SRR_cm21_1.fastq
-rw-r----- 1 gk1995 gk1995 4324777532 Apr 27 21:04 SRR_cm21_2.fastq

```

Next I used fastqc to check the raw reads for their quality. I placed the outputs into a separate scratch folder

```{bash}
$ nano ~/bash_files/fastqc.sh
_____________________
#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:30:00
#SBATCH --cpus-per-task=8
#SBATCH --nodes=2
#SBATCH --mem=8G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# $1 output folder

fastqc *.fastq -o $1
_____________________

$ ~/bash_files/fastqc.sh ~/scratch/fastqc_outputs/

```

I next rans multiQC to view the fastqc outputs. The python module needs to be loaded.
```{bash}
$ pip install --user multiqc
$ multiqc fastqc_outputs/
$ ll

total 98
drwxr-x--- 2 gk1995 gk1995   33280 Apr 28 12:52 fastqc_outputs
drwxr-x--- 2 gk1995 gk1995   33280 Apr 29 14:10 multiqc_data
-rw-r----- 1 gk1995 gk1995 1152757 Apr 29 14:10 multiqc_report.html
drwxr-x--- 2 gk1995 gk1995   33280 Apr 29 13:52 reads
```
#### The multiqc html report is found [HERE](https://rpubs.com/Greg1995/multiqc_cm21_raw)

## Trimming the raw reads

From the multiqc report, the reads appear to be trimmed for CM21. However, just in case I will run trimmomatic. It would be useful to incorperate trimming into the pipeline if additional strains are used. 

In this run I used the illumina adapters below. The correct adpaters need to be identified if raw reads were used. Typically The adapter sequences are identified by Multiqc.

```{bash}
$ mkdir trimmed_reads
$ cd trimmed_reads/

# the adapter seqeunces
$ nano trimmed_reads/TruSeq3-PE.fa
>PrefixPE/1
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PrefixPE/2
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT

nano ~/bash_files/trimmo_pair.sh
___________________

#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:15:00
#SBATCH --cpus-per-task=8
#SBATCH --nodes=1
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# $1 input_forward
# $2 input_reverse
# $3 individual name


java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE -phred33 -trimlog logfile $1 $2 $3_output_forward_paired.fastq $3_output_forward_unpaired.fastq $3_output_reverse_paired.fastq  $3_output_reverse_unpaired.fastq  ILLUMINACLIP:TruSeq3-PE.fa:2>

sacct -j $SLURM_JOB_ID --format=JobID%16,Submit,Start,Elapsed,NCPUS,ExitCode,NodeList%8
____________________

$ sbatch ~/bash_files/trimmo_pair.sh ~/scratch/reads/SRR_cm21_1.fastq ~/scratch/reads/SRR_cm21_2.fastq cm21
```

We will create a separate folder for the multiqc run of the trimmed files

```{bash}
$ mkdir trimmed_fastqc_outputs
$ cd trimmed_reads/
$ ~/bash_files/fastqc.sh ../trimmed_fastqc_outputs/
$ multiqc ~/scratch/trimmed_fastqc_outputs/

```

#### multiqc html report is found [HERE](https://rpubs.com/Greg1995/CM21_multiqc_trimmed)
Everything looks good. There are still no adapter sequences.

## To start alingning the CM21 Genome, Click [HERE](https://github.com/GregK10/GK_722_project/blob/main/3_Genome_Alignment_and_Consensus.md)





















