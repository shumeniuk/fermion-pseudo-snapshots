# Generating fermionic pseudo-snapshots
<img src="_resources/titlepage.png" width="600">

This repository contains subroutines for nested componentwise direct sampling (NCDS) of
occupation number states from pseudo free-fermion density matrices 
as they arise naturally in finite-temperature determinantal QMC (DQMC) simulations.

The code can be used in two forms, either 
1. as a stand-alone code, which uses as input the Green's functions which have been output and written 
   on disk during a previous DQMC run or 
2. directly called from a DQMC code so that fermionic snapshots are output "on the fly".    

The two use cases are descibed in the following.

Reading Green's function from a previous DQMC run:
--------------------------------------------------
Compile the code `sample_pseudo_DM.f90` with MPI support by calling 
```
make
```
This requires that you have `mpif90` installed. The resulting executable is called `sample_pseudo_DM`. 
Then, copy the provided test data for a 12x12 system at U/t=4, inverse temperature = 4 / t and half filling
```
cp _resources/U4.0_mu0.0_L12_beta4.0/Green_ncpu00000_*.dat .
```

The equal-time Green's function in real space for spin up and spin down is assumed to be stored in 
the files `Green_ncpuXXXXX_up.dat` and `Green_ncpuXXXXX_dn.dat`. The two files contain a stream 
of Green's function matrices for spin up and spin down in a synchronized fashion, i.e. the n-th 
Green's function in the "up-file" must be combined with the n-th Green's function in the "down-file". 
Successive Green's functions are separated by two empty lines.
The five-digit number code XXXXX labels DQMC output from different CPUs, parallelized via MPI, and 
the code for sampling of fermionic pseudosnapshots can be run with the same number of CPUs
by calling 
```
mpiexec.openmpi -np 1 ./sample_pseudo_DM
```
(We have only one set of Green's functions as test data, so we use only one CPU here.)

The number of snapshots to be generated can be set in the input file 
`simparams.in`.
```
filename = 'list_of_sitearrays.txt'   ! Here, a subset of sites for sampling can be selected (currently not used)
Nsites = 144                          ! Total number of sites, 12x12=144
max_HS_samples = 12                   ! Read a maximum number of Green's functions from the files Green_ncpuXXXXX_up(dn).dat  
Nsamples_per_HS = 10                  ! Generate `Nsamples_per_HS` number of occupation number snapshots per Green's function                   
skip = 0                              ! Discard the first `skip` number of Green's function 

```
With the above settings 120 snapshots will be generated per CPU. 
The snapshots will be written line-by-line into two synchronized files for spin up and spin down
called `Fock_samples_ncpuXXXXX_up.dat` and `Fock_samples_ncpuXXXXX_dn.dat`. The first two entries 
in each line are the sign and the reweighting factor. 
Note that the ordering of sites in a line is the same as the ordering used to write the 
Green's function matrix in real space.

The meaning of a line in the output files is as follows
```
1.0       1.0        0.0        1.253               1 0 0 1 1 0 0 1 1 1 1 0 0 1 0 1 ...
BSS_sign  Re(phase)  Im(phase)  reweighting_factor  occupation_vector(1:N_sites)
```
`BSS_sign` = sign(det(GF_up) * det(GF_dn)) is the sign of the current Green's function from which 
the sample was drawn. If the Hamiltionian is sign-problem free, it is equal to one. If there is a sign problem,
you need to calculate this one (the code does not do it right now). `Re(phase)` and `Im(phase)` give the 
phase of the pseudo-snapshot (note that for sampling in momentum space there is a *phase problem*). 
When calculating any quantity from the pseudo-snapshots they need to be weighted by `reweighting_factor`. 

Visualizing the generated pseudo-snapshots:
-------------------------------------------
After completing the above steps, we can visualize the pseudo-snapshots. The python script illustrates 
how to combine pseudo-snapshots for spin up and down. 
```
cp _resources/visualization/show_movie.py .
python show_movie.py
```


Interfacing with the QUantum Electron Simulation Toolbox (QUEST) DQMC code:
----------------------------------------------------------------------------
All relevant subroutines are in `NCDS_for_quest.F90`. 

From the open-source QUEST determinantal QMC code (-> http://quest.ucdavis.edu/index.html) 
the provided driver subroutine could be called like 
so in the subroutine `DQMC_Phy0_Meas`:


    call run_sample_pseudo_DM(G_up=G_up, G_dn=G_dn, BSS_sign_up=sgnup, BSS_sign_dn=sgndn,  sp_basis="real_space", &
           Nsamples_per_HS=20, outfile_basename="Fock_samples", &
           MPI_rank=qmc_sim%rank )


Then it will write the pseudo-napshots for spin-up and spin-down in two "synchronized" files together
with the sign (or phase factor) and reweighting factor.




If you use this code, please cite:
----------------------------------
```
@article{arXiv:2009.07377,
      title={Numerically exact quantum gas microscopy for interacting lattice fermions}, 
      author={Stephan Humeniuk and Yuan Wan},
      year={2020},
      eprint={2009.07377},
      archivePrefix={arXiv},
      primaryClass={cond-mat.quant-gas}
}
```


