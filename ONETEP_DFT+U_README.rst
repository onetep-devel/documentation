==========================================================
DFT+\ :math:`U`\ (+\ :math:`J`)
==========================================================

:Author: David D. O'Regan, Trinity College Dublin
	 
:Date:   July 2015


.. contents:: Table of Contents
   :depth: 3
   :local:
   :backlinks: none


DFT+\ :math:`U` is fully and self-consistently implemented in ONETEP,
together with a number of advanced ancillary functionalities. The method
is linear-scaling with respect to system size, exhibiting no systematic
tendency to slow convergence to the ground-state. DFT+\ :math:`U` in its
conventional fixed-projector form introduces only a small increase in
computational pre-factor with respect to the underlying
exchange-correlation functional [O-Regan2012]_.

A very short introduction to DFT+\ :math:`U`
============================================

DFT+\ :math:`U` [Anisimov1991]_, [Anisimov1997]_, [Dudarev1998]_, also
known as LDA+\ :math:`U` or LSDA+\ :math:`U`, is a method used to
improve the description of so-called strongly correlated materials
offered by DFT within conventional approximations for
exchange-correlation (XC) such as the LSDA and :math:`\sigma`-GGA,
quantitatively and even qualitatively. These functionals, based on the
locally-evaluated density and its gradients, can sometimes fail to
reproduce the physics associated with localised orbitals of :math:`3d`
and :math:`4f` character characteristic of conventionally-classed
strongly correlated materials, a category consisting of not only
first-row transition metals and their oxides, but also lanthanoid oxide
materials, and other materials such as certain magnetic semiconductors
and organometallic molecules.

Typically, the LDA and its extensions have the tendency to favour high-spin ground-states and to underestimate local magnetic
moments and the band gap in cases where it is related to electron localisation, e.g in charge-transfer or Mott insulators.
These failures stem from the inability of the approximate xc functionals to satisfy the flat-plane condition, 
which combines two constraints that the exact functional must meet: piecewise linearity with 
respect to electron number and the spin constancy constraint [Aron_J.Cohen2012]_

Put simply, if the system under study comprises open :math:`3d` or
:math:`4f` sub-shells, then there is a good chance that the LDA will
find a minimum energy configuration by partly occupying them, leading 
to an underestimation of the insulating gap and any associated magnetic order.

In DFT+U, the idea is to describe the strongly correlated electronic states using the Hubbard model by projecting the Kohn-Sham orbitals onto a localised basis, while the rest of the valence electrons 
are treated at the standard DFT level.

The general form for the total energy in DFT+U is given by: 

.. math::

   E_{\text{DFT}+U}[\rho(r)] = E_{\text{DFT}}[\rho(r)] + E_{\text{Hub}}\left[\{n_i\}\right] - E_{dc}\left[\{n_i\}\right]


Where :math:`E_{\text{DFT}}[\rho(r)]` is the energy at the standard DFT level, :math:`E_{\text{Hub}}\left[\{n_i\}\right]` is the Hubbard correction
and the last term :math:`E_{dc}\left[\{n_i\}\right]` is a double-counting correction for that contribution already
included in the LDA term.

The DFT+U correction term is usually thought of as an explicit
mean-field treatment of the exchange-correlation energy contributed by
the correlated sites (subspaces projected out with functions of
:math:`3d` and or :math:`4f` character).
There are many variants of the DFT+\ :math:`U` method, the flavour implemented in ONETEP is the
basis-set independent, rotationally invariant quadratic penalty
functional of Ref [Cococcioni2005]_, defined by the
additive energy correction. 

.. math::

   E_{DFT+U} \left[ n^{(I) (\sigma)} \right] =  \sum_{I \sigma} \frac{U^{(I)}}{2} \rm{Tr} 
   \left[  n^{(I) (\sigma)} \left( 1 -  n^{(I) (\sigma)} \right)\right].

Here, :math:`U` is an estimate of the scalar screened density-density
Coulomb repulsion between localised orbitals. 
The occupancy matrix of the correlated atomic site :math:`I`, for spin channel :math:`\sigma`, is
defined, in the case of orthonormal projector functions :math:`\lbrace \lvert \varphi^{(I)}_m \rangle \rbrace`, and density-matrix
:math:`\hat{\rho}^{(\sigma)}`, by


.. math::

   n^{(I)(\sigma)}_{m m'} = \langle \varphi_m^{(I)} \rvert \hat{\rho}^{(\sigma)} 
   \lvert \varphi_{m'}^{(I)} \rangle.



the DFT+\ :math:`U` method forces the on-site occupancy matrix to be idempotent.
For real matrices to be idempotent their eigenvalues have to be either 1 or 0, this means that the DFT+\ :math:`U` 
method penalises non-integer occupancy of these orbitals, tending to fill states with
occupancy greater than :math:`0.5` and empty states with occupancy less
than :math:`0.5`, as can be seen from the expression for the
DFT+\ :math:`U` potential

.. math::

   \hat{V}^{(\sigma)}_{DFT+U} = \sum_{I}  U^{(I)} 
    \lvert \varphi_m^{(I)} \rangle 
   \left( \frac{1}{2} \delta_{m m'} - n^{(I) (\sigma)}_{m m'} \right)  \langle 
   \varphi_{m'}^{(I)} \rvert .


The DFT+\ :math:`U` term may be considered as a correction which cancels
the contribution to the energy arising due to the spurious
self-interaction of a partially occupied
orbital [Cococcioni2005]_. In this case, the :math:`U`
parameter is the curvature of the total energy with respect to the
occupancy of the correlated manifold - which should be a piece-wise
linear curve were Janak’s theorem satisfied [Janak1978]_. 

The U parameter can be computed empirically or from linear-response theory (among other methods
such as constrained DFT) according to the prescription given in
Refs. [Cococcioni2005]_, [Kulik2006]_.

DFT+\ :math:`U` +\ :math:`J`
----------------------------

The complete rotationally invariant formulation of DFT+\ :math:`U` [Cococcioni2005]_ contains an explicit Hund's exchange term :math:`J`. 
In the simplified formulation, the :math:`J` term is either set to 0 or included in the :math:`U` parameter as
:math:`U_{eff} = U - J`. This is the so called Dudarev's model [Dudarev1998]_.

In this approach the exchange term works as an effective reduction to the U value for like-spin electrons.
A corrective term to include :math:`J` for anti-parallel spin is described in Ref. [Himmetoglu2011]_ and is given by

.. math::

   E_J[n^\sigma] = \sum_I \frac{J^I}{2} \text{Tr}[n^{I\sigma} n^{I-\sigma}]


Inclusion of this term leads to the so called  DFT+\ :math:`U` +\ :math:`J` method, which is  fully implemented in 
ONETEP and can be activated by
assigning a value to the :math:`J` term. 

Computing :math:`U` and :math:`J` from linear-response
=======================================================

The basic idea is to compare the response of the system to a perturbation in the DFT and in the DFT+U frameworks.
We start by defining the response function :math:`\chi`, which describes how the occupation of localized orbitals changes with respect to a shift in the potential acting on these orbitals:
The linear response method determines the Hubbard U parameter by comparing the response of the system to a perturbation in standard DFT and DFT+U frameworks.

We define the response function :math:`\chi` as: 

.. math::
   
   \chi = \frac{dn^{I\sigma}}{d\alpha}


where :math:`n` is the occupation matrix of the localized orbitals and :math:`\alpha` is a potential shift applied to these orbitals.

We compute two response functions:

- :math:`\chi_0`: the bare Kohn-Sham (KS) response (without U)
- :math:`\chi`: the interacting response (with U)

These are related by:

.. math::
   
   U = \chi^{-1} - \chi_0^{-1}

which allow us to compute :math:`U` 

In practice, we compute :math:`\chi_0` and :math:`\chi` by applying a small perturbation :math:`\alpha` to the system:


.. math::

    \chi_0 \approx \frac{\Delta n}{\Delta \alpha} 
    \text{(computed without U)}


.. math::

    \chi \approx \frac{\Delta n}{\Delta \alpha}
    \text{ (computed with a trial U)}  


The perturbation is applied by shifting the potential of the localized orbitals:

.. math::

   V_{ext}^{p} = V_{ext} + \alpha \sum_{m,m'}\lvert\varphi_{m'}^{(I)}\rangle\langle\varphi_m^{(I)}\rvert


We then iterate until self-consistency is achieved. 

This is done in a supercell as the perturbation should not interact with its periodic images.

This is the conventional linear response but it poses practial problems:
The response :math:`\chi_0` is usually computed via the first iteration
of the Kohn-Sham equations during a self-consistent field
(SCF) calculation; that is, the response is to be measured
following the initial charge redistribution introduced by the
perturbation but before the Kohn-Sham potential is updated.
This approach is impractical to implement in codes that
use a direct-minimization procedure of the total energy with
respect to the density, Kohn-Sham orbitals, or density matrix.

Another approach to compute :math:`U` and :math:`J` is known as minimum tracking method [Linscott2018]_

Minimum Tracking Method
-----------------------
The minimum tracking method is based on a reformulation of the response matrices
based on the ground state density of the perturbed
system. We can identify the interacting and noninteracting
response matrices as:

.. math::

   \chi_{IJ} = \frac{dn^I}{dv_\text{ext}^J},


.. math::

   (\chi_0)_{IJ} = \left[\frac{dn}{dv_\text{KS}}\left(\frac{dv_\text{KS}}{dv_\text{ext}}\right)^{-1}\right]_{IJ}


This allow us to prevent the practical issues from the conventional linear response.
This approach can also be extended to include the J exchange term (The response matrices now become Rank-Four tensor [Linscott2018]_), practically
this is done by modifying the perturbation by including an additional term (spin-splitting)

.. math::

   V_{ext}^{p} = V_{ext} + \beta \sum_{m,m'}\lvert\varphi_{m'}^{(I\uparrow)}\rangle\langle\varphi_m^{(I\uparrow)}\rvert-\lvert\varphi_{m'}^{(I\downarrow)}\rangle\langle\varphi_m^{(I\downarrow)}\rvert


Using NGWFs and projector self-consistency
==========================================

Any reasonable set of localised atomic-like functions may, in principle,
be used for the projectors defining the correlated subspaces in
DFT+\ :math:`U`; the choice is somewhat arbitrary and the description
“atomic orbitals" does not uniquely define them. One possible approach
is to use Wannier functions for the Kohn-Sham orbitals, so that the
correlated subspaces are proper subspaces of the Kohn-Sham Hilbert
space. Indeed, there is numerical evidence to suggest that Maximally
Localised Wannier Functions (MLWFs) [Marzari1997]_, [Souza2001]_,
in particular, provide a basis that maximises a particular measure of
the on-site Coulomb repulsion [Miyake2008]_, and MLWFs are
in common use as a minimal basis with which to construct tight-binding
models from first-principles.

In ONETEP, a set of variationally-optimised nonorthogonal generalised
Wannier functions (NGWFs) are generated as a by-product of total-energy
minimisation. NGWFs exhibit some similar properties to MLWFs and other
flavours of localised Wannier functions, and, for example, can be used
to calculate finite-difference response properties in a similar
way [O-Regan2012-2]_. As they are conveniently available in
ONETEP, we have made it possible to re-use the NGWFs from the end of a
ground-state calculation as a set of Hubbard projectors with which to
define the DFT+\ :math:`U` correction. For this, it was necessary to
develop a tensorially-consistent formulation of DFT+\ :math:`U` in order
to accommodate nonorthogonal projector
functions [O-Regan2011]_; projector nonorthogonality
for a given subspace is automatically compensated for.

In order to ensure that NGWFs with appropriate symmetry are chosen as
Hubbard projectors for a given atom, those :math:`n` NGWFs
:math:`\lvert \phi_\alpha \rangle` that maximise :math:`\sum^n_{m,\alpha }\langle \varphi_m  \rvert  \phi^\alpha \rangle \langle \phi_\alpha \rvert \varphi_m \rangle`, for a given set of
:math:`n` hydrogenic orbitals :math:`\lvert \varphi_m \rangle`, defined
in the ``hubbard`` block, are selected for the task. 

The keyword ``hubbard_max_iter``, (defaulting to :math:`0`), sets the task to
``HUBBARDSCF``, which performs a self-consistency cycle over the Hubbard
projectors, demonstrated in
Refs. [O-Regan2010]_, [O-Regan2011]_. 

The density from one minimisation is re-used at the beginning of the next, and setting
``hubbard_max_iter`` to :math:`2` one can carry out a DFT+\ :math:`U`
calculation using the LDA NGWFs as projectors.

The keywords ``hubbard_energy_tol``, ``hubbard_conv_win``, and
``hubbard_proj_mixing`` are used to manage the Hubbard projector
self-consistency cycle. For convergence, the ground state energy must
deviate less than ``hubbard_energy_tol`` (defaulting to
:math:`10^{-8}Ha`) from one ``HUBBARDSCF`` iteration to the next, over
``hubbard_conv_win`` (defaulting to :math:`2`) iterations. A fraction
``hubbard_proj_mixing`` (defaulting to :math:`0.0`) of the previous
Hubbard projectors may be mixed with the new ones in order to accelerate
the procedure, although this has never been found to be necessary.
Setting ``hubbard_proj_mixing`` to a negative value causes the
projectors to be read in from a ``.tightbox_hub_projs`` file, for
restarting a ``HUBBARDSCF`` calculation or for a variety of
post-processing tasks.

How to activate DFT+\ :math:`U` in ONETEP
=========================================

In order to activate the DFT+\ :math:`U` functionality, the **hubbard**
block is added to the input file. For example, in the case of a system
containing iron and cerium atoms incorrectly described by the
exchange-correlation functional, which we suspect could benefit from the
DFT+\ :math:`U` correction to improve the description of localisation,
we might use the ``hubbard`` block:

.. code-block:: none
   
   % block hubbard
     Fe1   2   4.0   0.0  -10.0   0.00   1.0
     Fe2   2   4.0   0.0  -10.0   0.00  -1.0
     Ce1   3   6.0   0.0  -10.0   0.50   0.0
   % endblock hubbard

The columns of the ``hubbard`` block are described as follows:

1. **Species Label**

   The species to apply the DFT+\ :math:`U` correction to.
   In this example Fe1, Fe2 and Ce1.


2. **Angular Momentum:** :math:`l`

   The angular momentum of the projectors which the Hubbard correction is applied to.
   In this example :math:`l=2` for Fe1 and Fe2 and :math:`l=3` for Ce1.
   Conventionally, the radial quantum number :math:`r=l+1` is used to generate atom-centred atomic
   projectors, so that :math:`l=2` gives :math:`3d` orbitals,
   :math:`l=3` gives :math:`4f` orbitals etc. 
   
   (please get in contact if you need to use a :math:`r \ne l+1` combination, or multiple sub-shells per atom).

3. **Hubbard** :math:`U` **value**
   
   The value of the Hubbard :math:`U` for this sub-shell, in
   electron-volts. 
   
4. **Hund's exchange** :math:`J` **value**
   
   The value of the Hund’s exchange :math:`J` for this sub-shell, in
   electron-volts. 
   The rotationally invariant exchange corrective term
   described in detail in Ref. [Himmetoglu2011]_ (The so called DFT+\ :math:`U` +\ :math:`J`) is fully
   implemented in ONETEP (including forces etc), and activated for any
   :math:`J \ne 0`. 

5. **Effective Charge Z and Projectors type**
   
   - Case 1: :math:`\mathbf{ Z < 0}` (Default)
      
      A subset of the orbitals generated by solving the atomic problem subject to the pseudopotential for the species in question are chosen (in which case
      the projectors form a subset of the initial guesses for the ONETEP
      NGWFs); here the magnitude of the negative Z makes no difference. 
   
   - Case 2: :math:`\mathbf{ Z > 0}` 
      
      The projectors are generated in the form of solutions to the
      hydrogenic Schrödinger equation. In this case :math:`\mathbf{Z}`
      is the effective charge divided by the ratio of effective
      masses used to generate projectors. A good guess for this number might be the Clementi-Raimondi effective charge, 
      tabulated in Refs. [Clementi1963]_, [Clementi1967]_, and the choice of
      radial profile does matter [O-Regan2010]_. 
   
   In both cases, the projectors are effectively renormalised within an atom-centred sphere with the same radius as the NGWFs on that atom.

6. **The** :math:`\alpha` **prefactor**
   
   An additional potential acting on the subspace in question, the
   prefactor :math:`\alpha` is here entered in electron-volts. This is
   needed, for example, in order to locally vary the potential in order
   to determine the value of :math:`U` which is consistent with the
   screened response in the system with linear-response
   theory [Cococcioni2005]_, [Kulik2006]_, or to break a
   spatial symmetry, such as in a mixed-valence system. In the example
   given, we are additionally penalising the occupancy on cerium
   :math:`4f` atomic orbitals.

7. **The spin-splitting factor**
   
   The spin-splitting factor, in electron-volts, which is deducted from
   the :math:`\alpha` factor for the spin-up channel and added to
   :math:`\alpha` for the spin-down channel. In the example shown here
   we’re promoting spin-up magnetisation for iron atoms :math:`Fe1`, and
   spin-down for :math:`Fe2`. This can be very useful for appropriately
   breaking magnetic symmetries in antiferromagnetic solids or
   open-shell singlet molecules, or for estimating the magnetic
   susceptibility or exchange coupling.

   **N.B.** Users may find the DFT+\ :math:`U` functionality useful in
   cases of systems even when the DFT+\ :math:`U` correction is not
   needed (setting the all :math:`U` parameters to zero does not disable
   the functionality). The implementation offers a very inexpensive
   method for carrying out carefully-defined atom-centred atomic
   population analysis, or breaking symmetries in spin or charge ordered
   systems.


DFT+\ :math:`U` keywords
=========================


.. table:: DFT+\ :math:`U` keywords
   :name: dft+u_keywords

   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   |Keyword                          | Type     |Default                    | Description                                        |
   +=================================+==========+===========================+====================================================+
   | ``HUBBARDSCF``                  | Task     |      —                    | | Activate a projector-self-consistent DFT+U       |
   |                                 |          |                           | | calculation.                                     |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+ 
   | ``HUBBARDSCF_ON_THE_FLY``       | Logical  |  False                    | | Activate a non-variational on-the-fly form of    |
   |                                 |          |                           | | projector self-consistency in DFT+U or cDFT, in  |
   |                                 |          |                           | | which the projectors are updated whenever the    |
   |                                 |          |                           | | NGWFs are. task : HUBBARDSCF is then not needed. |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+                    
   |``HUBBARD_CONV_WIN``             | Integer  | ``2``                     | | The minimum number of Hubbard projector update   |
   |                                 |          |                           | | steps satisfying the incremental energy tolerance| 
   |                                 |          |                           | | HUBBARD_ENERGY_TOL required for convergence      |
   |                                 |          |                           | | in task : HUBBARDSCF.                            |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   |``HUBBARD_ENERGY_TOL``           | Physical | ``1.0E-8 Ha``             | | The maximum incremental energy change between    |
   |                                 |          |                           | | Hubbard projector update steps allowed for       |
   |                                 |          |                           | | converge in task : HUBBARDSCF.                   |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_FUNCTIONAL``          | Real     | ``1``                     | | The form of DFT+U energy term used. Contact      |
   |                                 |          |                           | | developers if you need to try something beyond   |
   |                                 |          |                           | | the default.                                     |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_MAX_ITER``            | Integer  | ``10``                    | | The maximum allowed number of Hubbard projector  |
   |                                 |          |                           | | update steps taken in a projector self-consistent|
   |                                 |          |                           | | DFT+U or cDFT calculation in task : HUBBARDSCF.  | 
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_NGWF_SPIN_THRESHOLD`` | Physical |``2.0E-5 Ha``              | | The incremental change in energy, in             |
   |                                 |          |                           | | total-energy minimisation, at which any          |
   |                                 |          |                           | | spin-splitting (Zeeman) type term in DFT+U is    |
   |                                 |          |                           | | switched off, and the minimisation history reset.| 
   |                                 |          |                           | | Useful for breaking open-shell,                  | 
   |                                 |          |                           | | antiferromagnetic, or charge-density             |
   |                                 |          |                           | | wave symmetries.                                 |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_PROJ_MIXING``         | Real     | ``0.0``                   | | The fraction of previous Hubbard projector to    |
   |                                 |          |                           | | mix with new for projector self-consistent DFT+U |
   |                                 |          |                           | | or cDFT in task : HUBBARDSCF.                    |
   |                                 |          |                           | | Not found to be necessary.                       |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_READ_PROJECTORS``     | Logical  |``False``                  | | Read Hubbard projectors from .tightbox_hub_projs |
   |                                 |          |                           | | file in restart calculations involving DFT+U.    |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+
   | ``HUBBARD_TENSOR_CORR``         | Integer  | ``1``                     | | The form of correction used to correct for any   |
   |                                 |          |                           | | nonorthogonality between Hubbard projectors.     |
   |                                 |          |                           | | Contact developers if you need to try something  |
   |                                 |          |                           | | other than the default "tensorial" correction.   |
   +---------------------------------+----------+---------------------------+----------------------------------------------------+



Tutorials
=========

Example of the use of DFT+U in Hematite, a strongly correlated system
   - https://tutorials.onetep.org/T9_hematite_dftu.html

Example on how to compute U and J from linear response
   - To be added

Compatibility
=============

The DFT+\ :math:`U` functionality is fully compatible with almost all
other parts of the ONETEP code, such as listed below, since it simply
involves an additional term in the Hamiltonian and ionic forces. Please
get in touch first if you would like to use a more exotic combination of
these functionalities:

#. Total-energy minimisation and ionic forces

#. Geometry optimisation, molecular dynamics and phonon calculations

#. All other functionals including hybrids and Van der Waals functionals

#. Implicit solvation

#. The PAW formalism and ultrasoft pseudopotentials

#. Constrained DFT

#. Local density of states (including a correlated subspace
   decomposition)

#. Natural bond orbital calculations

#. Conduction-band optimisation and Fermi’s Golden Rule spectra

#. Calculations of changes in electric polarisation

#. Time-dependent DFT

#. Electronic transmission calculations

The extension of the DFT+\ :math:`U` implementation to cluster Dynamical
mean-field theory has also been implemented in ONETEP; for an example of
its capabilities see Ref. [Weber2012]_.

References
==========


.. [O-Regan2012] \ D. D. O’Regan, N. D. M. Hine, M. C. Payne and A. A. Mostofi, Phys. Rev. B **85**, 085107 (2012). https://doi.org/10.1103/PhysRevB.85.085107

.. [Anisimov1991] \ J. Z. V. I. Anisimov and O. K. Andersen, Phys. Rev. B **44**, 943 (1991). https://doi.org/10.1103/PhysRevB.44.943

.. [Anisimov1997] \ V. I. Anisimov, F. Aryasetiawan, and A. I. Liechtenstein, J. Phys.: Condens. Matter **9**, 767 (1997). https://iopscience.iop.org/article/10.1088/0953-8984/9/4/002

.. [Dudarev1998] \ S. L. Dudarev, Phys. Rev. B **57**, 3 (1998). https://doi.org/10.1103/PhysRevB.57.1505

.. [Aron_J.Cohen2012] \ A. J. Cohen, P. Mor-Sanchez and W. Yang, Chem. Rev. 2012, 112, 289–320. https://doi.org/10.1021/cr200107z

.. [Cococcioni2005] \ M. Cococcioni and S. de Gironcoli, Phys. Rev. B **71**, 035105 (2005). https://doi.org/10.1103/PhysRevB.71.035105

.. [Janak1978] \ J. F. Janak, Phys. Rev. B **18**, 12 (1978). https://doi.org/10.1103/PhysRevB.18.7165

.. [Kulik2006] \ H. J. Kulik, M. Cococcioni, D. A. Scherlis and N. Marzari, Phys. Rev. Lett. **97**, 103001 (2006). https://doi.org/10.1103/PhysRevLett.97.103001

.. [Himmetoglu2011] \ B. Himmetoglu, R. M. Wentzcovitch, and M. Cococcioni, Phys. Rev. B,\ **84**, 115108 (2011). https://doi.org/10.1103/PhysRevB.84.115108

.. [Clementi1963] \ E. Clementi and D.L. Raimondi, J. Chem. Phys. **38**, 2686 (1963). https://doi.org/10.1063/1.1733573

.. [Clementi1967] \ E. Clementi, D.L. Raimondi, and W.P. Reinhardt, J. Chem. Phys. **47**, 1300 (1967). https://doi.org/10.1063/1.1712084

.. [O-Regan2010] \ D. D. O’Regan, N. D. M. Hine, M. C. Payne and A. A. Mostofi, Phys. Rev. B **82**, 081102 (2010). https://doi.org/10.1103/PhysRevB.82.081102

.. [Weber2012] \ C. Weber, D. D. O’Regan, N. D. M. Hine, M. C. Payne, G. Kotliar and P. B. Littlewood, Phys. Rev. Lett. **108**, 256402 (2012). https://doi.org/10.1103/PhysRevLett.108.256402

.. [Marzari1997] \ N. Marzari and D. Vanderbilt, Phys. Rev. B **56**, 12847 (1997). https://doi.org/10.1103/PhysRevB.56.12847

.. [Souza2001] \ I. Souza, N. Marzari and D. Vanderbilt, Phys. Rev. B **65**, 035109 (2001). https://doi.org/10.1103/PhysRevB.65.035109

.. [Miyake2008] \ T. Miyake and F. Aryasetiawan, Phys. Rev. B **77**, 085122 (2008). https://doi.org/10.1103/PhysRevB.77.085122

.. [O-Regan2012-2] \ D. D. O’Regan, M. C. Payne, and A. A. Mostofi, Phys. Rev. B **85**, 193101 (2012). https://doi.org/10.1103/PhysRevB.85.193101

.. [O-Regan2011] \ D. D. O’Regan, M. C. Payne and A. A. Mostofi, Phys. Rev. B **83**, 245124 (2011). https://doi.org/10.1103/PhysRevB.83.245124

.. [Linscott2018] \ E.B. Linscott, D. J. Cole, M. C. Payne, D. D. O'Regan, Phys. Rev. B **98**, 235157 (2018). https://doi.org/10.1103/PhysRevB.98.235157
