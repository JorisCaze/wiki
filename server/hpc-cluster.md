# Memento to setup an HPC cluster

1. Add admin user to super user group
2. Setup SSH with SSH key access only and deactivate root authentication (see ssh.md)
3. Separate home partition from root partition (see quota.md)
4. Setup user quota (see quota.md)
5. Install Open-MPI
6. Install Slurm job scheduler (see slurm.md)
7. Install mail tools (see mail.md)