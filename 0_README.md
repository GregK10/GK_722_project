# GK_722_project
My Bio722 individual project code

Welcome to my 722 project Github page. For my project, I looked to identify and investigate microsatellite (SSR) loci from reference and a lab strain of A. fumigatus

In this project, I used Sharcnet to conduct my analyses. I initially attempted to use Brian's cluster, but I had difficulties getting PERF to work. The authors primary suggestion was to use the pip install, but we are discouraged from using it on the cluster. The alternate methods described in the PERF readme did not work on Brian's cluster so I switched to running the program on Sharcnet. 

Since I am running my scripts on Sharcnet using the Graham cluster, they require specific identifiers so the cluster can use the jobs. For more intensive scripts, such as those used for trimmoatic or genome alignment, I added addition commandds so I would be put into a queue.

For my less computationally intense scripts, I detailed

```{bash}
#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```
### To proceed to running PERF, click [HERE](https://github.com/GregK10/GK_722_project/blob/main/1_Perf_and_the_reference_genome.md)
