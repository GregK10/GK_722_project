Accidently went to far? Click [HERE](https://github.com/GregK10/GK_722_project/blob/main/0_README.md) to go back to the readme

## Initial run of PERF

For the first part of my project, I tested the algorithm PERF on the _A. fumigatus_ reference strain Af293. PERF is a recently developed algorith that identifies SSR motifs from within DNA sequences.

I needed to install the dependencies required for PERF to run.
```{bash}
$ mkdir 722_project ; cd 722_project/
$ module load python/3
$ pip install biopython --no-index
$ pip install regex --no-index
$ pip install tqdm --no-index
$ pip install perf_ssr
```


Next, I downloaded the reference genome for _A. fumigatus_, Af293
```{bash}
$ wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/655/GCF_000002655.1_ASM265v1/GCF_000002655.1_ASM265v1_genomic.fna.gz
$ gunzip GCF_000002655.1_ASM265v1_genomic.fna.gz ; mv GCF_000002655.1_ASM265v1_genomic.fna.gz ref_af293.fa
```

I then ran PERF to test if it works and identifies the microsatellite loci present within the Af293 genome. I created a directory for where my bash scripts

```{bash}
$ mkdir ~/bash_files 
$ nano ~/bash_files/perf.sh

_______________
#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

PERF -input $1 --output $2  --analyse
_______________________________

$ chmod +x ~/bash_files/perf.sh # I only included this once as it has to be done for every script i created
```


The script will take the input file (reference genome) and output in a filename I designate. --analyse creates a html file that summarizes the microsatellite information created by PERF in fasta format. The table below decribes the data generated (taken from [Perf GitHub](https://raw.githubusercontent.com/RKMlab/perf/master/README.md)). The output can be specified in fastq format as well.

| S.No | Column | Description |
|:----:| ------ | ----------- |
| 1 | Chromosome | Chromosome or Sequence Name as specified by the first word in the FASTA header |
| 2 | Repeat Start | 0-based start position of SSR in the Chromosome |
| 3 | Repeat Stop | End position of SSR in the Chromosome |
| 4 | Repeat Class | Class of repeat as grouped by their cyclical variations |
| 5 | Repeat Length | Total length of identified repeat in nt |
| 6 | Repeat Strand | Strand of SSR based on their cyclical variation |
| 7 | Motif Number | Number of times the base motif is repeated |
| 8 | Actual Repeat | Starting sequence of the SSR irrespective of Repeat class and strand|

Below is bash script and the output of the run.

```{bash}
$ ~/bash_files/perf.sh ref_Af293.fa perf_out

 Using length cutoff of 12
 Processing NC_007201.1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████  ███████████████████████████████████████████████████| 8/8 [00:16<00:00,  2.05s/it]

 Generating HTML report. This may take a while..

 HTML report successfully saved to perf_out.html
 
$ head perf_out
NC_007194.1     0       123     AACCCT  123     +       20      CCTAAC
NC_007194.1     1750    1762    ACAT    12      -       3       TGTA
NC_007194.1     3815    3827    ATATCC  12      +       2       CATATC
NC_007194.1     4453    4466    AGCCTG  13      +       2       GAGCCT
NC_007194.1     5984    5996    AT      12      +       6       TA
NC_007194.1     11452   11464   AGAGCC  12      -       2       GCTCTG
NC_007194.1     12317   12332   ACTGCC  15      -       2       TGGCAG
NC_007194.1     12650   12663   AACGAC  13      -       2       GTCGTT
NC_007194.1     15208   15220   ACGATG  12      -       2       ATCGTC
NC_007194.1     17437   17450   AACGC   13      -       2       GTTGC
```
As you can see above, there are many hexamer repeats that are two repeats long. These repeat types are very common within the Af293 genome.


### The HTML files that were generated can be accessed [HERE](https://rpubs.com/Greg1995/Af293)


I also tried to identify where each repeat was located, either intergenic, or within introns or exons. The ```--annotate``` flag requires a gene annotation file in either GFF or GTF format. Therefore, the GFF and GTF file for Af293 was downloaded. For simplicity, I downloaded the files to my local machine and then I used to mobaxterm terminal to upload it to my Sharcnet account. I then unzipped both folders using tar and then unzipped the GTF and GFF files in both folders using gunzip. 

```{bash}
$ cd ~/722_project

$ tar -xvf genome_assemblies_genome_gff.tar 
$ gunzip ncbi-genomes-2021-04-28/GCA_000002655.1_ASM265v1_genomic.gff.gz
$ mv ncbi-genomes-2021-04-28/GCA_000002655.1_ASM265v1_genomic.gff af293.GFF

$ tar -xvf genome_assemblies_genome_gtf.tar 
$ gunzip ncbi-genomes-2021-04-28/GCA_000002655.1_ASM265v1_genomic.gtf.gz
$ mv ncbi-genomes-2021-04-28/GCA_000002655.1_ASM265v1_genomic.gff af293.GTF
```

I then made a new script that included the annotation file
```{bash}
nano ~/bash_files/perf_anno.sh 
_________________
#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

PERF -input $1 --output $2 --anotate $3  --analyse
_______________________________
```

Unfortunately, this gave me trouble to get it to work. After trouble shooting, I was not able to identify what was causing the problem. I tried using both GTF and GFF files, however neither worked. When using the GTF file, the error below occurred

```{bash}
$ ~/bash_files/perf_anno.sh ref_Af293.fa GTF_test af293.GFF gtf
Using length cutoff of 12
Processing NC_007201.1: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 8/8 [00:15<00:00,  1.93s/it]

Generating annotations for identified repeats..

  0%|                                                                                                                                                                                                           | 0/21138 [00:00<?, ?it/s]
Traceback (most recent call last):
  File "/home/gk1995/.local/bin/PERF", line 8, in <module>
    sys.exit(main())
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/core.py", line 162, in main
    ssr_native(args, length_cutoff=args.min_length)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/core.py", line 106, in ssr_native
    fasta_ssrs(args, repeats_info)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/rep_utils.py", line 253, in fasta_ssrs
    annotate(args)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/annotation.py", line 291, in annotate
    minStartIndex += minIndex
UnboundLocalError: local variable 'minIndex' referenced before assignment
```

When using a GFF file, a more informative error occurred. 

```{bash}
$ ~/bash_files/perf_anno.sh ref_Af293.fa GTF_test af293.GFF

Using length cutoff of 12
Processing NC_007201.1: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 8/8 [00:15<00:00,  1.95s/it]

GeneKeyError:
The attribute "gene" is not among the attributes for gene. Please select a different one.
The available ones are [ID, Name, end_range, gbkey, gene_biotype, locus_tag, old_locus_tag, partial, start_range]

```

To try to appeases the error, I updated my script as follows. I added the --anno-format flag to specify GFF file and the --gene-key flag to specify "ID" shown in the above error and what is seen in the GFF file. 

```{bash}
nano ~/bash_files/perf_anno.sh

#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

PERF -input $1 --output $2 --anotate $3 --anno-format $4 --gene-key $5 --analyse

```

Unfortunately again, the GFF run with ID stated resulted in a error in the PERF script and did not generate the SSR annotation file. I also tried some of the attributes copied directly from the GFF file, but none of them worked. Each would complete the original SSR call but failed after attempting the annotation. I also could not find a solution online or in the github page. I also tried attributes I found in the GTF file as well.
Attributes I used included: Name, "Name", "ID, ID=gene, gene, "gene".

```{bash}
$ ~/bash_files/perf_anno.sh ref_Af293.fa GFF_test af293.GFF gff ID

Using length cutoff of 12
Processing NC_007201.1: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 8/8 [00:14<00:00,  1.85s/it]

Generating annotations for identified repeats..

  0%|                                                                                                                                                                                                           | 0/21138 [00:00<?, ?it/s]
Traceback (most recent call last):
  File "/home/gk1995/.local/bin/PERF", line 8, in <module>
    sys.exit(main())
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/core.py", line 162, in main
    ssr_native(args, length_cutoff=args.min_length)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/core.py", line 106, in ssr_native
    fasta_ssrs(args, repeats_info)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/rep_utils.py", line 253, in fasta_ssrs
    annotate(args)
  File "/home/gk1995/.local/lib/python3.8/site-packages/PERF/annotation.py", line 291, in annotate
    minStartIndex += minIndex
UnboundLocalError: local variable 'minIndex' referenced before assignment

```

If the annotation completed successfully, It would have provided me with the information in the table below. I would have used the information generated to identify where the microsatellites were located and compare the differences between strains. Table taken from [Perf GitHub](https://raw.githubusercontent.com/RKMlab/perf/master/README.md))

| S.No | Column | Description |
|:----:| ------ | ----------- |
| 9 | Gene name | Name of the closest gene |
| 10 | Gene Start | Start position of gene in the Chromosome |
| 11 | Gene Stop | End position of gene in the Chromosome |
| 12 | Strand | The strand orientation of the gene |
| 13 | Genomic annotation | Annotation of the repeat w.r.t to the gene. Possible annotations are {Genic, Exonic, Intronic, Intergenic} |
| 14 | Promoter annotation | If repeat falls in the promoter region of the closest gene. The default promoter region is 1Kb upstream and downstream of TSS. |
| 15 | Distance from TSS | Distance of the repeat from the TSS of the gene. |


### Up next, I obtained the raw reads from the A. fumgiatus strain cm21 and used a trimmomatic script to trim the raw reads. Click [HERE](https://github.com/GregK10/GK_722_project/blob/main/2_Obtaining_trimming_and_QC_of_reads.md) to continue.


