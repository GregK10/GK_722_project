Want to go back to [obtaining trimming and QC of reads?](https://github.com/GregK10/GK_722_project/blob/main/2_Obtaining_trimming_and_QC_of_reads.md)

## Genome Alignment and Consensus

Next up, we want to align the cm21 genome to the af293 reference genome. I will be using bwa for aligning

First, the reference genome is indexed so it can ran by ```bwa mem```. I have included ```bwa index``` into my bash script for simplicity, however, it should be run separately as multiple index runs are not needed.

```{bash}
$ nano ~/bash_files/bwa.sh

#!/bin/bash
#SBATCH --account=######
#SBATCH --time=1:30:00
#SBATCH --cpus-per-task=16
#SBATCH --nodes=2
#SBATCH --mem=60G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

module load bwa
module load samtools
module load picard

bwa index ~/722_project/ref_Af293.fa

bwa mem -t 8 ~/722_project/ref_Af293.fa $1 $2  | samtools view -bSh > ~/scratch/cm21.bam

samtools sort ~/scratch/cm21.bam -o ~/scratch/cm21_sorted.bam

samtools index cm21_sorted.bam

samtools flagstat cm21_sorted.bam

java -jar $EBROOTPICARD/picard.jar MarkDuplicates I=cm21_sorted.bam O=cm21.markdup.bam M=cm21_mark_metrics.txt

samtools index cm21.markdup.bam

samtools flagstat cm21.markdup.bam > cm21.markdup_mstats.txt



sacct -j $SLURM_JOB_ID --format=JobID%16,Submit,Start,Elapsed,NCPUS,ExitCode,NodeList%8

```

Instead of the java line that was used to mark duplicates, I initally attempted to use samtools markdup. This command requires samtool fixmate to be run prior, but I was unable to get it to work with my data. I therefore elected to the Picard command to mark dupicates instead

I then checked how well the cm21 reads mapped to the Af293 reference genome by viewing the ```cm21.markdup_mstats.txt``` file

```{bash}
$ more cm21.markdup_mstats.txt

6818907 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
25187 + 0 supplementary
162338 + 0 duplicates
6084343 + 0 mapped (89.23% : N/A)
6793720 + 0 paired in sequencing
3396860 + 0 read1
3396860 + 0 read2
5921452 + 0 properly paired (87.16% : N/A)
6022446 + 0 with itself and mate mapped
36710 + 0 singletons (0.54% : N/A)
78774 + 0 with mate mapped to a different chr
54096 + 0 with mate mapped to a different chr (mapQ>=5)

```
## Creating the Consensus

With the summary text looking good as all reads passed, I then alligned the cm21 gennome to the af293 genome to generate a consensus fasta file. I used the ```vcf-consensus``` command from vcftools to create the consensus sequence and then converted it to a fasta file. PERF requires either fasta for fastq files for the script to run.

```{bash}
$ cat ~/722_project/ref_Af293.fa | vcf-consensus cm21SNPs.vcf.gz > cm21_con.fa
$ head -n 5 cm21_con.fa

>NC_007194.1 Aspergillus fumigatus Af293 chromosome 1, whole genome shotgun sequence
cctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaac
cctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaac
ccttTAGGCAACTGCAGCcTCAAACCgGATTTGGATGGGCCgCACGCGTGCTAGGTTTCC
```

The name is still incorrect as I forgot to incorporate the ```bwa mem -R``` flag to tell the ```bwa mem``` to rename head of the file to cm21. To fix this issue, I used the ```sed``` command below to replace Af293 with cm21

```{bash}
$ sed -i 's/Af293/cm21/g' cm21_con.fa
$ head -n 5 cm21_con.fa

>NC_007194.1 Aspergillus fumigatus cm21 chromosome 1, whole genome shotgun sequence
cctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaac
cctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaaccctaac
ccttTAGGCAACTGCAGCcTCAAACCgGATTTGGATGGGCCgCACGCGTGCTAGGTTTCC
TGGTTCTTGGAACGACATTGTTCTCACCTAGTGTGATGAGCGTTCGTCAAGTTCCAAAGC
```

### Next up, I will run PERF on the CM21 genome. Click [here](https://github.com/GregK10/GK_722_project/blob/main/4_PERF_and_plotting_results.md) to continue.
