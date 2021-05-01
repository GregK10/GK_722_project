Go back to [Genome Alignment and Consensus](https://github.com/GregK10/GK_722_project/blob/main/3_Genome_Alignment_and_Consensus.md)

## Running PERF with CM21
With the cm21 consensus sequence obtained, I then ran the PERF script to identify the SSR alleles. I used the same bash script I used for af293
 
```{bash}
$ nano ~/bash_files/perf.sh

#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:03:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

PERF --input $1 --output $2 --analyse 
```
 
Running the script on the CM21 genome
```{bash}
$ ~/bash_files/perf.sh cm21_con.fa cm_perf_con

Using length cutoff of 12
Processing NC_007201.1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 8/8 [00:15<00:00,  1.96s/it]

Generating HTML report. This may take a while..

HTML report successfully saved to cm_perf_con_v2.html
```

This generated the html summary report see [HERE](https://rpubs.com/Greg1995/CM21_PERF)

Similar to the run with af293, I was not able to use the annotation file to identify if the SSRs are within annotated locations in the genome.  

```{bash}
$ ~/bash_files/perf_anno.sh cm21_con.fa cm_perf_con_GFF af293.GFF gff ID
Using length cutoff of 12
Processing NC_007201.1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 8/8 [00:15<00:00,  1.94s/it]

Generating annotations for identified repeats..

  0%|                                                                                                                                                                                  | 0/20990 [00:00<?, ?it/s]
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
## Plotting Results in R with ggplot

Within the html outputs for both strains, there was on option to download the csv that details the SSR position and type. I downloaded the file, provided headers and saved it is as a csv.

```{r}
library(ggplot2)
cm21 <- read.csv("D:/School/Bio_722/cm21_microsats.csv", header = TRUE)
head(cm21)
```

Using the file, I created a ggplot to visualize the SSR density throughout the genome. I used x = end to prevent any issues with the SSR position = 0. I then faceted it by the chromosome. I formatted the graph so it would look nice :)

```{r}
ggplot(data = cm21, aes(x = end, fill = chromosome)) +
        geom_histogram(binwidth = 100000) +
        facet_wrap(vars(chromosome), scales = "free_x") +
        scale_x_continuous(labels = function(x) format(x/10000)) +
        xlab("position (10kb)") +
        theme_classic()
```
 
Now for the same plot script but instead with af293
```{r}
af293 <- read.csv("D:/School/Bio_722/af293_microsats.csv", header = TRUE)
head(af293)
ggplot(data = af293, aes(x = end, fill = chromosome)) +
        geom_histogram(binwidth = 100000) +
        facet_wrap(vars(chromosome), scales = "free_x") +
        scale_x_continuous(labels = function(x) format(x/10000)) +
        xlab("position (10kb)") +
        theme_classic()
```

As seen in the figure, both strains have similar SSR density throughout their chromosomes.

I can also filter the rows so that specific SSR repeats lengths are selected. Here is an example for repeats that are lengths = 4 for Af293.

```{r}
library(dplyr)
ssr3_af293 <- dplyr::filter(af293, nchar(af293$repeat.) == 4)
head(ssr3_af293)
ggplot(data = ssr3_af293, aes(x = end, fill = chromosome)) +
        geom_histogram(binwidth = 100000) +
        facet_wrap(vars(chromosome), scales = "free_x") +
        scale_x_continuous(labels = function(x) format(x/10000)) +
        xlab("position (10kb)") +
        theme_classic()
```
And here is an example for repeats that are lengths = 4 for CM21.
```{r}
library(dplyr)
ssr3_cm21 <- dplyr::filter(cm21, nchar(cm21$repeat.) == 4)
head(ssr3_cm21)
ggplot(data = ssr3_cm21, aes(x = end, fill = chromosome)) +
        geom_histogram(binwidth = 100000) +
        facet_wrap(vars(chromosome), scales = "free_x") +
        scale_x_continuous(labels = function(x) format(x/10000)) +
        xlab("position (10kb)") +
        theme_classic()
``` 
 
 
This concludes my work-flow for this project. I was greatly interested in identifying and comparing the presence and length of SSRs within intronic, exonic and promoter regions of the genome between different strains. Unfortunately I did not generate this data. I may need to contact the authors and inquire what may be the issue. 
In the future when I will be using more than one strain, I would optomize the scripts to include ```for``` loops to run multiple strains.  

The PERF authors recently updated their scripts, potentially creating a bug for the annotation step. 

### Thank you for viewing my project. To go back to the readme file, click [HERE](https://github.com/GregK10/GK_722_project/blob/main/0_README.md).
