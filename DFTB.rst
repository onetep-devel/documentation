==========================================
Density Functional Tight Binding in ONETEP
==========================================

:Author: Arihant Bhandari, University of Southampton, United Kingdom
:Author: Jacek Dziedzic, University of Southampton, United Kingdom
:Author: Chris-Kriton Skylaris, University of Southampton, United Kingdom

:Date: December 2022

.. role:: raw-latex(raw)
   :format: latex
..

.. raw:: latex

   \maketitle

.. contents:: Table of Contents
   :depth: 3
   :local:
   :backlinks: none


Introduction to Density Functional Tight Binding (DFTB)
=======================================================

Density Functional Tight binding models have been derived by a Taylor expansion
of the DFT energy functional in terms of the electron density and truncation
up to a certain order in the expansion [Foulkes1989]_. Within DFTB, 
the following eigenvalue equations are solved via diagonalization:

.. math::
  :label: Kohn_Sham_equation

   \begin{aligned}
      H_{\alpha\beta}M^{\beta}_{\,i} = S_{\alpha\beta} M^{\beta}_{\,i} \epsilon_i,  
   \end{aligned}

where :math:`H_{\alpha\beta}` is the hamiltonian matrix, :math:`S_{\alpha\beta}` 
is the overlap matrix, :math:`M^{\beta}_{\,i}` are the orbital coefficients, and 
:math:`\epsilon_i` are energy eigenvalues. The Hamiltonian is built from
parameters. [Elstner1998]_ proposed a self consistent charge (SCC) extension 
to the traditional DFTB approach, which optmizes the atomic charges
self consistently. Henceforth, a series of SCC DFTB methods have been developed [Gaus2014]_. 
Recently, non-SCC methods have undergone a revival because of their speed and applicability 
to large systems [Bannwarth2020]_. E.g. GFN0 is one such method  
where atomic charges are found using a charge equilibriation scheme [Pracht2019]_. 
The total energy in GFN0 also includes zeroth order terms such as dispersion, repulsion, 
electrostatic interactions and short range basis correction. 

We have implemented the GFN0 method taking the advantage of
linear-scaling capabilities of ONETEP. 
Here we include D2 dispersion correction [Grimme2006]_ instead of D4 [Caldeweyher2019]_.
The standard ensemble-DFT subroutines are used for diagonalization and
calculation of electronic energies and forces. The parameter files
are available in [utils-devel]_. 


Keywords
========

-  ``dftb: T/F`` 
  | [Boolean, default ``dftb: F``]. 
  | If true, it enables DFTB calculations.

-  ``dftb_method: GFN0`` 
  | [Text, default ``dftb_method: GFN0``]. 
  | Variant of the DFTB method, only GFN0 has been implemented at the moment. 

-  ``dftb_method_param_file: file address`` 
  | [Text, default ``dftb_method_param_file: param_gfn0-xtb.txt``]. 
  | Path to the parameter file. A specimen file is supplied in utils-devel/dftb folder. 

-  ``dftb_common_param_file: file address`` 
  | [Text, default ``dftb_common_param_file: param_gfn_common.txt``]. 
  | Path to the file for common GFN parameters. A specimen file is supplied in utils-devel/dftb folder. 

-  ``dftb_bc: O O O / P P P`` 
  | [Boolean, default ``dftb_bc: P P P``]. 
  | Boundary conditions. Only fully open (O O O) or full periodic (P P P) are implemented. 

-  ``dftb_coord_cutoff: 50.0 ang`` 
  | [Real Physical, default ``dftb_coord_cutoff: 40.0 bohr``]. 
  | Cutoff distance for truncating interactions for calculating coordination numbers. Refer to eq. (10) of [Pracht2019]_. 

-  ``dftb_rep_cutoff: 50.0 ang`` 
  | [Real Physical, default ``dftb_rep_cutoff: 40.0 bohr``]. 
  | Cutoff distance for truncating interactions for calculating repulsion energy. Refer to eq. (3) of [Pracht2019]_.

-  ``dftb_srb_cutoff: 20.0 ang`` 
  | [Real Physical, default ``dftb_srb_cutoff: 14.14 bohr``]. 
  | Cutoff distance for truncating interactions for calculating short-range basis correction energy. Refer to eq. (4) of [Pracht2019]_.

-  ``dftb_ewald_parameter: 5.0 ang-1`` 
  | [Real Physical, default ``dftb_ewald_parameter: -1.0 bohr-1``]. 
  | If positive, this value is used as the parameter for Ewald summation for periodic electrostatic interactions. 
   Otherwise, the optimum parameter for ewald summation is calculated on the
   fly.

-  ``dftb_cartesian_ngwfs: T/F`` 
  | [Boolean, default ``dftb_cartesian_ngwfs: F``]. 
  | If true, the program uses Cartesian Gaussian orbitals, otherwise the program uses spherical orbitals as basis. 
   Note that the currently implemented GFN0 method has been developed for a basis of spherical orbitals and may 
   give incorrect results with Cartesian orbitals.

-  ``dftb_overlap_analytical: T/F`` 
  | [Boolean, default ``dftb_overlap_analytical: T``]. 
  | If false, elements of the overlap are calculated via integrals on grid, otherwise analytically. 
   Note that the gradients of overlap matrix on grid are not yet implemented.


Acknowledgement
===============

We would like to thank Loukas Kollias, Denis Kramer and John R. Owen for useful discussions.

.. [Foulkes1989] \ W. Matthew C. Foulkes, Roger Haydock, *Phys. Rev. B* **1989**, 39, 12520, https://doi.org/10.1103/PhysRevB.39.12520

.. [Elstner1998] Marcus Elstner et. al., *Phys. Rev. B* **1998**, 58, 7260, https://doi.org/10.1103/PhysRevB.58.7260

.. [Gaus2014] Michael Gaus, Qiang Cui, Marcus Elstner, "Density functional tight binding: application to organic biological molecules", *WIREs Comput. Mol. Sci.* **2014**, 4, 49, https://doi.org/10.1002/wcms.1156

.. [Bannwarth2020] Christoph Bannwarth et. al., "Extended tight-binding quantum chemistry methods", *WIREs Comput. Mol. Sci.* **2021**, 11, 1, https://doi.org/10.1002/wcms.1493

.. [Pracht2019] Philipp Pracht, Eike Caldeweyher, Sebastian Ehlert, Stefan Grimme, "A robust non-self-consistent tight-binding quantum chemistry method for large molecules", *ChemRxiv* **2019**, https://doi.org/10.26434/chemrxiv.8326202.v1

.. [Grimme2006] Stefan Grimme, "Semi-empirical GGA-type density functional constructed with a long-range dispersion correction", *J. Comput. Chem.* **2006**, 27, 1787, https://doi.org/10.1002/jcc.20495

.. [Caldeweyher2019] Eike Caldeweyher et. al., "A generally applicable atomic-charge dependent London dispersion correction", *J. Chem. Phys.* **2019**, 150, 154122, https://doi.org/10.1063/1.5090222

.. [utils-devel] https://github.com/onetep-devel/utils-devel 

