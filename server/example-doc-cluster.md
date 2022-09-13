# Documentation of the Cid HPC server

Welcome to the Cid HPC cluster!

1. Architecture

1.1. HPC

This cluster is made of a single-node that serves as both a connection front-end and a computation node.

Here are given some informations:

- OS   : Ubuntu 20.04
- CPUs : 52 Intel Xeon Gold 6230R configured as 
        - 48 CPUs are reserved for computation purposes
        - 4 CPUs are reserved for users
- GPU  : Nvidia Quadro RTX 6000
- RAM  : 252 Go DDR4

1.2. Storage

Each user has a home directory 250 Go of storage, which can be extended to 300 Go during 7 days. 
For specific storage needs, please contact the administrator of this server at cid.hpc@gmail.com.

2. Run a simulation

To run a simulation/job, Slurm job scheduler is provided on this cluster. 
Its use is mandatory, for more information about Slurm check the official website.

2.1. Submit a job

Job can be submitted using a sbatch script:

$ sbatch ./job.sh

Example of sbatch script:

*******************

#!/bin/sh

# Slurm batch directives

#SBATCH -J jobName
#SBATCH -N 1
#SBATCH --ntasks-per-node=5
#SBATCH -t 07-00:00:00
#SBATCH -o ./%x-%j.out
#SBATCH -e ./%x-%j.err
# #SBATCH --mail-type=BEGIN,END
# #SBATCH --mail-user=user.name@mail.fr

# Run ECOGEN CFD code on 5 CPUs (ntask-per-node)
mpirun "/home/username/codes/ecogen/ECOGEN" "/home/username/codes/ecogen/"

*******************

2.2. Manage jobs

To see current submitted jobs on the server, you can use one the following command:

$ sinfo -o "%.8P %.5a %.6D %.6t %.8N %.9B %.11A %.14C %.6D %.8O"
$ sinfo -o "%n %a %C"

2.3. Compile code

Since this server is a really small HPC cluster, it is requested to compile big codes by submitting a sbatch job.

3. Available tools

Some softwares/librairies are installed for users such as:
- Gmsh 3.0.6
- ParaView 5.7.0, 5.8.1, 5.9.1
- ECOGEN 3.0.1
- Open-MPI 4.4.1 (available in PATH directly)

They can be accessed in /opt/ if the user is a member of the group related to the software/library. 

For specific software installation or to get access to an existing one if not already possible, please contact the administrator of this server at cid.hpc@gmail.com.

4. Rules

- Jobs must be run with Slurm. Usage of nohup or interactive session to run jobs is forbbiden, any process concerned will be terminated without any warnings.
- Each user has a limited storage capacity of 250 Go, it can be extended to 300 Go during 7 days, after this period data can be deleted by the admin without any warnings.

If you have any problems, please contact the administrator of this server at cid.hpc@gmail.com.
