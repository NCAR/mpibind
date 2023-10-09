# mpibind
## MPI binding utilities for the NCAR Derecho system
The mpibind utility is a universal tool for binding MPI applications on Derecho. It supports binding of hybrid MPI + OpenMP CPU applications as well as MPI-enabled GPU applications. It supports a "naked" MPI launch style where all relevant MPI rank, OpenMP thread, and GPU layout information is taken from the PBS select statement, and is not explicitly required on the MPI launch line. It also supports multiple MPI libraries utilizing identical launch syntax between Cray-MPICH, Intel MPI, MVAPICH, and OpenMPI. The binding does not rely on MPI library binding syntax, but instead uses the standard Linux tools like lscpu, numactl, and nvidia-smi.

## Usage
mpibind \<application\>

e.g.

mpibind ./a.out

Additional binding information will be written to a logfile (mpibind.log) in the execution directory.

## CPU Binding Example
### Example PBS script

```shell
#!/bin/bash -l
#PBS -A <project code>
#PBS -N hybrid_cpu
#PBS -j oe
#PBS -q main
#PBS -l walltime=00:30:00
#PBS -l select=2:ncpus=128:mpiprocs=2:ompthreads=4

module --force purge
module load ncarenv/23.06 craype/2.7.20 cce ncarcompilers/1.0.0 cray-mpich/8.1.25

export TMPDIR=/glade/derecho/scratch/$USER/tmp
mpibind ./hybrid_cpu.exe
```

### Example binding log file
```
Chunk info
  2:ncpus=128:mpiprocs=2:ompthreads=4:mem=250gb:Qlist=cpu:ngpus=0
-- -- -- --
MPI exec line:
  mpiexec -n 4 -ppn 2 --cpu-bind none -env OMP_NUM_THREADS=4 cpu_bind ./hybrid_cpu.exe
-- -- -- --
Binding Report:
rank: 0, cores: 0-3
rank: 1, cores: 64-67
rank: 2, cores: 0-3
rank: 3, cores: 64-67
```

### Example Application Output
```
Total threads: 16
  Rank 0 has 4 threads
  Rank 1 has 4 threads
  Rank 2 has 4 threads
  Rank 3 has 4 threads
--- --- --- --- ---
MPI Task   0, OpenMP thread 0 of 4 (cpu 0 of dec0146)
              OpenMP thread 1 of 4 (cpu 1 of dec0146)
              OpenMP thread 2 of 4 (cpu 2 of dec0146)
              OpenMP thread 3 of 4 (cpu 3 of dec0146)
MPI Task   1, OpenMP thread 0 of 4 (cpu 64 of dec0146)
              OpenMP thread 1 of 4 (cpu 65 of dec0146)
              OpenMP thread 2 of 4 (cpu 66 of dec0146)
              OpenMP thread 3 of 4 (cpu 67 of dec0146)
MPI Task   2, OpenMP thread 0 of 4 (cpu 0 of dec0150)
              OpenMP thread 1 of 4 (cpu 1 of dec0150)
              OpenMP thread 2 of 4 (cpu 2 of dec0150)
              OpenMP thread 3 of 4 (cpu 3 of dec0150)
MPI Task   3, OpenMP thread 0 of 4 (cpu 64 of dec0150)
              OpenMP thread 1 of 4 (cpu 65 of dec0150)
              OpenMP thread 2 of 4 (cpu 66 of dec0150)
              OpenMP thread 3 of 4 (cpu 67 of dec0150)
```

## GPU Binding Example
### Example PBS script
```shell
#!/bin/bash -l
#PBS -A <project code>
#PBS -q main
#PBS -l select=2:ncpus=64:mpiprocs=4:ngpus=4
#PBS -N mpi_gpu
#PBS -l walltime=00:30:00

module --force purge
module load ncarenv/23.06 craype/2.7.20 nvhpc/23.5 ncarcompilers/1.0.0 cray-mpich/8.1.25 cuda/11.7.1

mpibind ./mpi_gpu.exe
```

### Example binding log file
```
Chunk info
  2:ncpus=64:mpiprocs=4:ngpus=4:ompthreads=1:mem=487gb:Qlist=a100
-- -- -- --
MPI exec line:
  mpiexec -n 8 -ppn 4 --cpu-bind none -env OMP_NUM_THREADS=1 gpu_bind ./mpi_gpu.exe
-- -- -- --
Binding Report:
rank: 0, cores: 0, CUDA DEVICE: 3, UUID: GPU-ae2bd804-5000-28d4-d125-c99414389583
rank: 1, cores: 16, CUDA DEVICE: 2, UUID: GPU-ee2c7651-9b8f-0787-01d6-160554b5df79
rank: 2, cores: 32, CUDA DEVICE: 1, UUID: GPU-b28b8c09-72ac-03e3-d3a4-6629b7dbdab0
rank: 3, cores: 48, CUDA DEVICE: 0, UUID: GPU-29a80481-04d1-94f3-b3bc-b8854dccde49
rank: 4, cores: 0, CUDA DEVICE: 3, UUID: GPU-13d320e1-452b-126d-d9e2-ad50e6b3c3a1
rank: 5, cores: 16, CUDA DEVICE: 2, UUID: GPU-20a97009-be16-753a-8c2b-a3ebc6c2ab21
rank: 6, cores: 32, CUDA DEVICE: 1, UUID: GPU-05431eac-bb16-3d25-bd3b-8e756a532c1e
rank: 7, cores: 48, CUDA DEVICE: 0, UUID: GPU-92b31c0a-c818-5302-29b5-4b5e9bc7e515
```

### Example Application Output
```
----- ----- -----
Using 8 MPI Ranks and GPUs
----- ----- -----
Message before GPU computation: xxxxxxxxxxxx
----- ----- -----
rank 0 on host deg0067, CPU: 0, GPU: 0, UUID: GPU-ae2bd804-5000-28d4-d125-c99414389583
rank 1 on host deg0067, CPU: 16, GPU: 0, UUID: GPU-ee2c7651-9b8f-0787-01d6-160554b5df79
rank 2 on host deg0067, CPU: 32, GPU: 0, UUID: GPU-b28b8c09-72ac-03e3-d3a4-6629b7dbdab0
rank 3 on host deg0067, CPU: 48, GPU: 0, UUID: GPU-29a80481-04d1-94f3-b3bc-b8854dccde49
rank 4 on host deg0069, CPU: 0, GPU: 0, UUID: GPU-13d320e1-452b-126d-d9e2-ad50e6b3c3a1
rank 5 on host deg0069, CPU: 16, GPU: 0, UUID: GPU-20a97009-be16-753a-8c2b-a3ebc6c2ab21
rank 6 on host deg0069, CPU: 32, GPU: 0, UUID: GPU-05431eac-bb16-3d25-bd3b-8e756a532c1e
rank 7 on host deg0069, CPU: 48, GPU: 0, UUID: GPU-92b31c0a-c818-5302-29b5-4b5e9bc7e515
----- ----- -----
 Message after GPU computation: Hello World!
----- ----- -----
 TEST SUCCESSFUL
----- ----- -----
