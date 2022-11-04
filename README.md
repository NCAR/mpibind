# mpibind
## MPI binding utilities for the NCAR Gust system
The mpibind utilities are a set of universal tools for binding hybrid MPI + OpenMP applications. They support a "naked" MPI launch style where all relevant rank and thread layout information is taken from the PBS select statement, and not explicitly required on the MPI launch line. They are also MPI library agnostic and support the same launch syntax between Cray-MPICH and OpenMPI. The binding does not rely on MPI library binding syntax, but instead uses the standard Linux numactl tool.

## Usage
mpibind \<binding style\> \<application\>

e.g.

mpibind balanced ./a.out

Additional binding information will be written to a logfile (mpibind.log) in the execution directory.

### Binding Styles
1. balanced

   Splits MPI ranks as evenly as possible between sockets, and binds threads compactly to adjacent CPUs.

### Example PBS script

```shell
#!/bin/bash -l
#PBS -A SCSG0001
#PBS -q main
#PBS -l select=1:ncpus=128:mpiprocs=2:ompthreads=4+1:ncpus=128:mpiprocs=4:ompthreads=3
#PBS -N donde
#PBS -l walltime=01:10:00

module purge
module load ncarenv/22.10 craype/2.7.17 cce/14.0.3 ncarcompilers/0.7.1 cray-mpich/8.1.19

mpibind balanced ./donde.cray > donde.out
```

### Example Output
```
Total threads: 20
  Rank 0 has 4 threads
  Rank 1 has 4 threads
  Rank 2 has 3 threads
  Rank 3 has 3 threads
  Rank 4 has 3 threads
  Rank 5 has 3 threads
--- --- --- --- ---
MPI Task   0, OpenMP thread 0 of 4 (cpu 0 of gu0001)
              OpenMP thread 1 of 4 (cpu 1 of gu0001)
              OpenMP thread 2 of 4 (cpu 2 of gu0001)
              OpenMP thread 3 of 4 (cpu 3 of gu0001)
MPI Task   1, OpenMP thread 0 of 4 (cpu 64 of gu0001)
              OpenMP thread 1 of 4 (cpu 65 of gu0001)
              OpenMP thread 2 of 4 (cpu 66 of gu0001)
              OpenMP thread 3 of 4 (cpu 67 of gu0001)
MPI Task   2, OpenMP thread 0 of 3 (cpu 0 of gu0002)
              OpenMP thread 1 of 3 (cpu 1 of gu0002)
              OpenMP thread 2 of 3 (cpu 2 of gu0002)
MPI Task   3, OpenMP thread 0 of 3 (cpu 3 of gu0002)
              OpenMP thread 1 of 3 (cpu 4 of gu0002)
              OpenMP thread 2 of 3 (cpu 5 of gu0002)
MPI Task   4, OpenMP thread 0 of 3 (cpu 64 of gu0002)
              OpenMP thread 1 of 3 (cpu 65 of gu0002)
              OpenMP thread 2 of 3 (cpu 66 of gu0002)
MPI Task   5, OpenMP thread 0 of 3 (cpu 67 of gu0002)
              OpenMP thread 1 of 3 (cpu 68 of gu0002)
              OpenMP thread 2 of 3 (cpu 69 of gu0002)
```

### Example log file
```
Chunk info
  1:ncpus=128:mpiprocs=2:ompthreads=4
  1:ncpus=128:mpiprocs=4:ompthreads=3
-- -- -- --
MPI exec line:
  mpiexec -n 2 -ppn 2 --cpu-bind none -env OMP_NUM_THREADS=4 balanced ./donde.cray : -n 4 -ppn 4 --cpu-bind none -env OMP_NUM_THREADS=3 balanced ./donde.cray
```
