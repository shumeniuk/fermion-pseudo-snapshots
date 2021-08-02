# sample-pseudoDM-fortran

<img "_resources/titlepage.png" width="400">

Subroutines for nested componentwise direct sampling for occupation number states from pseudo free fermion density matrices 
as they arise naturally in finite-temperature determinantal QMC simulations.

From the QUEST determinantal QMC code (-> http://quest.ucdavis.edu/index.html) 
the provided driver subroutine could be called like 
so in the subroutine `DQMC_Phy0_Meas`:


    call run_sample_pseudo_DM(G_up=G_up, G_dn=G_dn, BSS_sign_up=sgnup, BSS_sign_dn=sgndn,  sp_basis="real_space", &
           Nsamples_per_HS=20, outfile_basename="Fock_samples", &
           MPI_rank=qmc_sim%rank )


Then it will write the snapshots for spin-up and spin-down in two "synchronized" files together
with the sign and reweighting factor. Further documentation can be found in the source code itself.

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


