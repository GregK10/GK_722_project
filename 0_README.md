# GK_722_project
My Bio722 individual project code

Welcome to my 722 project Github page. For my project, I looked to identify and investigate microsatellite (SSR) loci in _A. fumigatus_ using the reference Af293 and the lab strain CM21

In this project, I used Sharcnet to conduct my analyses. I initially attempted to use Brian's cluster, but I had difficulties getting PERF to work. The authors primary suggestion was to use the pip install, but we are discouraged from using it on the cluster. The alternate methods described in the PERF readme did not work on Brian's cluster so I switched to running the program on Sharcnet. 

Since I am running my scripts on Sharcnet using the Graham cluster, they require specific identifiers so the cluster can use the jobs. For my less computationally intense scripts, I removed the sacct option as they took minimal computation effort. When running many strains potentially later in my thesis project, I would add the ```sacct``` line. For computationally heavier scripts, such as those used for trimmoatic or genome alignment, I kept the ```sacct``` line and preceeded my script call with sbatch to be put into a queue.

```{bash}
#!/bin/bash
#SBATCH --account=######
#SBATCH --time=00:10:00
#SBATCH --cpus-per-task=6
#SBATCH —nodes=2
#SBATCH --mem=15G
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

sacct -j $SLURM_JOB_ID --format=JobID%16,Submit,Start,Elapsed,NCPUS,ExitCode,NodeList%8
```
### To proceed to running PERF, click [HERE](https://github.com/GregK10/GK_722_project/blob/main/1_Perf_and_the_reference_genome.md)
