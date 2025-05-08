======================================
Setting up k-points and spin in ONETEP
======================================

:Author: Chengcheng Xiao, Miguel Escobar Azor and Nicholas D.M. Hine
:Date:   May 2025


Spin-polarisation in ONETEP
===========================

TBC


k-points and Brillouin zone sampling in ONETEP
==============================================

ONETEP now supports k-point sampling in the Brillouin zone. By utilising the k-point
sampling technique, the periodicity of the system can be exploited to reduce the
computational cost of the calculation and small systems (from unit cells upwards) can be
efficiently simulated.

The k-point sampling technique is implemented in two modes: the plane-wave (PW)
mode and the tight-binding (TB) mode. The default behaviour in ONETEP is to assume
the system is periodic but that the simulation cell is large enough that k-point
sampling is not required. In order to launch a calculation on a simulation cell where
this is not the case, so k-point sampling is required, the user must first choose
between these modes as appropriate to the periodicity of their system, and then
specify a grid of k-points on which the calculation is to be performed.


Plane-wave mode (PW) mode
=========================

In the plane-wave (PW) mode, the k-dependent Bloch functions are expressed using
cell-periodic functions and and the Bloch phase factor. The cell-periodic
functions are further expanded in terms of NGWFs.

Specifically, to satisfy the minimum image convention (i.e., no NGWF should 
overlap with its periodic images), the localisation constraint is relaxed along 
the directions where k-point sampling is applied. Since the NGWFs are expressed
using psinc functions, by relaxing the localisation constraint, NGWFs are
essentially expressed using plane waves (hence the name "plane-wave mode").

Theory
------
The Schrödinger equation (or in the case of DFT, the Kohn-Sham equation) for a
Bloch eigenstate :math:`\psi_{n,\mathbf{k}}` is given by

.. math::
   :label: SE

   \hat{H} \psi_{n,\mathbf k} (\mathbf r) = \epsilon_{n,\mathbf k} \psi_{n,\mathbf k} (\mathbf r).

where :math:`\mathbf k` labels the k-point, :math:`n` the band number.
Immediately we can see that we need to solve this equation separately at each k-point,
hence we need to have seperate representations of any quantities arising from
:math:`\psi_{n,\mathbf k}` for each k-point.

The most generic form of the Bloch functions is written as,

.. math::
   :label: bloch

   \psi_{n,\mathbf k} (\mathbf r) = e^{i\mathbf k \cdot \mathbf r} u_{n,\mathbf k} (\mathbf r),

where :math:`u_{n,\mathbf k} (\mathbf r)` is a function with the periodicity of the unit cell,
i.e., 

.. math::
   u_{n,\mathbf k} (\mathbf r) = u_{n,\mathbf k} (\mathbf r + \mathbf R),

and :math:`\mathbf R` is a Bravais lattice vector. 

Inserting :eq:`bloch` into :eq:`SE`, we get 

.. math::
   \hat{H} e^{i\mathbf k \cdot \mathbf r} u_{n,\mathbf k} (\mathbf r) = \epsilon_{n,\mathbf k} e^{i\mathbf k \cdot \mathbf r} u_{n,\mathbf k} (\mathbf r) .

Premultiplying both sides by :math:`e^{i\mathbf k \cdot \mathbf{r}}`, we get

.. math::
   :label: SE2

   e^{-i\mathbf k \cdot \mathbf r} \hat{H} e^{i\mathbf k \cdot \mathbf r} u_{n,\mathbf k} (\mathbf r) = \epsilon_{n,\mathbf k} u_{n,\mathbf k} (\mathbf r).

If we introduce the k-dependent Hamiltonian operator :math:`\hat{H}(\mathbf k)`,

.. math::
   \hat{H}(\mathbf k) = e^{-i\mathbf k \cdot \mathbf r} \hat{H} e^{i\mathbf k \cdot \mathbf r},

:eq:`SE2` can then be written as

.. math::
   :label: SE3

   \hat{H}(\mathbf k) u_{n,\mathbf k} (\mathbf r) = \epsilon_{n,\mathbf k} u_{n,\mathbf k} (\mathbf r). 

In ONETEP we use a set of k-dependent NGWFs (:math:`\{ \phi_\alpha^{\mathbf k}\}`) as our support functions,
so the cell-periodic part of the Bloch wavefunction :math:`u_{n,\mathbf{k}}` can be written as

.. math::
   u_{n,\mathbf k} (\mathbf r)  = \sum_{\alpha} c_{n,\mathbf k}^{\alpha} \phi_{\alpha}^{\mathbf k} (\mathbf r),

where :math:`c_{n,\mathbf k}^{\alpha}` is a non-unitary rotation matrix that 
rotates the NGWFs (see TB note).

Expanding :math:`u_{n,\mathbf k}` using in terms of NGWFs (which, as we will see later
will need to be "extended" NGWFs along periodic directions), :eq:`SE3` can be expressed as

.. math::
   :label: SE4

   \hat{H}(\mathbf k) u_{n,\mathbf k} (\mathbf r)= \sum_{\beta} c_{n, \mathbf k}^{\beta}\hat{H}(\mathbf k) \phi_{\beta}^{\mathbf k} (\mathbf r)  =  \epsilon_{n,\mathbf k} \sum_{\beta} c_{n,\mathbf k}^{\alpha} \phi_{\beta}^{\mathbf k} (\mathbf r)

If we multiply both sides of :eq:`SE4` by :math:`\phi_{\alpha}^{\mathbf k *}(\mathbf r)`, and integrate over :math:`\mathbf r`, we get

.. math::
   :label: SE5

   \sum_{\beta} c_{n, \mathbf k}^{\beta} \int \phi_{\alpha}^{\mathbf k *}(\mathbf r)\hat{H}(\mathbf k) \phi_{\beta}^{\mathbf k} (\mathbf r) d\mathbf r= \sum_{\beta} c_{n, \mathbf k}^{\beta} \epsilon_{n,\mathbf k} \int \phi_{\alpha}^{\mathbf k *}(\mathbf r) \phi_{\beta}^{\mathbf k} (\mathbf r) d\mathbf r. 

In matrix notation, :eq:`SE5` can be written as

.. math::
   :label: SE6

   \sum_{\beta} H_{\alpha \beta}(\mathbf k) c_{n, \mathbf k}^{\beta} = \epsilon_{n,\mathbf k} \sum_{\beta} c_{n, \mathbf k}^{\beta} S_{\alpha \beta}^{\mathbf k}, 

where the Hamiltonian matrix :math:`H` elements are:

.. math::
   H_{\alpha \beta} (\mathbf k) = \int \phi_{\alpha}^{\mathbf k *}(\mathbf r)\hat{H}(\mathbf k) \phi_{\beta}^{\mathbf k} (\mathbf r) d\mathbf r. 

and the overlap matrix :math:`S` elements are:

.. math::
   S_{\alpha \beta} (\mathbf k) = \int \phi_{\alpha}^{\mathbf k *}(\mathbf r)\phi_{\beta}^{\mathbf k} (\mathbf r) d\mathbf r. 

:eq:`SE6` is the generalized Kohn-Sham equation under the basis of 
:math:`\phi`. If :math:`\phi` were simply plane waves, :math:`\mathbf{S}` would be
the identity, and we could return to the canonical expression of the Kohn-Sham
equation in the plane-wave basis:

.. math::
   \sum_{\beta} H_{\alpha \beta}(\mathbf k) c_{n, \mathbf k}^{\beta} = \epsilon_{n,\mathbf k}  c_{n, \mathbf k}^{\alpha}

It is worth nothing that in the plane-wave basis the explicit k-dependence only exists in the 
Hamiltonian matrix and the k-index of eigenvectors is the result of solving such
a k-dependent Hamiltonian. In the NGWF basis, there is also k-dependence in the overlap matrix.

The Hamiltonian can be expanded into three terms: the kinetic energy term
[:math:`T_{\alpha \beta}(\mathbf{k})`],
the potential term (including local potential from atomic cores and exchange
correlation terms) and the non-local terms from pseudopotentials.

Kinetic energy term
^^^^^^^^^^^^^^^^^^^

The matrix elements of the kinetic energy operator under the psinc basis are given by

.. math::
   T_{\alpha \beta}(\mathbf{k}) =\int d^3 r \phi_\alpha^{\mathbf{k} *}(\mathbf{r}) 
   \mathrm{e}^{-\mathrm{i} \mathbf{k} \cdot \mathbf{r}} (-\frac{\nabla_{\mathbf r}^2}{2}) \mathrm{e}^{\mathrm{i} \mathbf{k} \cdot \mathbf{r}} \phi_\beta^{\mathbf{k}}(\mathbf{r}), 

Using psinc functions :math:`D_i(\mathbf r)`:

.. math::
   D_{i} (\mathbf r) = \frac{1}{N}\sum_p e^{i\mathbf G_p \cdot (\mathbf r - \mathbf r_i)}, 

The kinetic matrix elements becomes

.. math::
   \begin{aligned}
   T_{\alpha \beta}(\mathbf{k}) &=\int d^3 r \phi_\alpha^{\mathbf{k} *}(\mathbf{r}) 
   \mathrm{e}^{-\mathrm{i} \mathbf{k} \cdot \mathbf{r}} (-\frac{\nabla_{\mathbf r}^2}{2}) \mathrm{e}^{\mathrm{i} \mathbf{k} \cdot \mathbf{r}} \phi_\beta^{\mathbf{k}}(\mathbf{r}) \\
   &= -\frac{V}{2N^2} \sum_{ij} c^{\mathbf k *}_{j \alpha} c^{\mathbf k}_{i \beta} \sum_{p} (-|\mathbf G_p + \mathbf k|^2) e^{i\mathbf G_p (\mathbf r_j - \mathbf r_i)}. \\ 
   &= \frac{V}{2N^2} \sum_{ij} c^{\mathbf k *}_{j \alpha} c^{\mathbf k}_{i \beta} \sum_{p} (|\mathbf G_p + \mathbf k|^2) e^{i\mathbf G_p (\mathbf r_j - \mathbf r_i)}.
   \end{aligned}

where we see that we only need to add k vector to the :math:`\mathbf{G}` vectors
to make the kinetic energy matrix k-dependent.

Local potential, hartree and exchange-correlation energy terms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The k-dependent local potential energy matrix is given by:

.. math::
   \begin{aligned}
   V_{\mathrm{LHXC},\alpha\beta}(\mathbf{k}) &= \int d^3 r \phi_\alpha^{\mathbf{k} *}(\mathbf{r})
   e^{-\mathrm{i} \mathbf{k} \cdot \mathbf{r}} V_\mathrm{LHXC}(\mathbf{r})
   e^{\mathrm{i} \mathbf{k} \cdot \mathbf{r}}
   \phi_\beta^{\mathbf{k}}(\mathbf{r}) \\
   &= \int d^3 r \phi_\alpha^{\mathbf{k} *}(\mathbf{r}) V_\mathrm{LHXC}(\mathbf{r})
   \phi_\beta^{\mathbf{k}}(\mathbf{r}) \\
   \end{aligned}

And we see that no explicit k-dependence is needed in the local
potential term.

Non-local potential term
^^^^^^^^^^^^^^^^^^^^^^^^

The k-dependent non-local potential term is given by:

.. math::
   V_{\mathrm{NL},\alpha\beta}(\mathbf{k}) = \sum_{I,lm}
   \left<\phi_\alpha^{\mathbf{k}}|e^{-\mathrm{i} \mathbf{k} \cdot \mathbf{r}}|\xi^I_{lm}\right> V_{lm}^I(\mathbf{k})
   \left<\xi^I_{lm}|e^{\mathrm{i} \mathbf{k} \cdot \mathbf{r}}|\phi_\beta^{\mathbf{k}}\right>

and we see that the phase factors are to be augmented to the non-local
projectors so that the non-local potential term is k-dependent.

Brillouin zone sampling
-----------------------

To find the ground state of the system, we need to sample the Brillouin zone by
performing a summation over the results of different k-points.
Specifically, we need to perform k-point average over all energy components and
the charge density.

The k-point sampling is performed using the Monkhorst-Pack scheme where kpoints
along a spcific direction are generated (in the first Brillouin zone) by

.. math::
   \frac{2r-q-1}{2q}

where :math:`q` is the number of k-points in each direction and :math:`r` is the
k-point index. An optional shift of half a grid cell can be added so
that :math:`\Gamma` point is included in the sampling.

There are two ways to define the k-point sampling in ONETEP:

1. Automatic generation using the Monkhorst-Pack scheme.:

   ::
   
      kpoint_grid_shift : 0 1 1
      kpoint_grid_size : 3 6 6

   which indicates that the size of the k-point grid is 3 along a direction, 6 
   along b direction (shifted by half a grid distance) and 6 along c direction
   (also shifted by half a grid distance).


2. Manual generation using the k-point coordinates:

   ::
   
      %block kpoints_list
          0.5000000000    0.50000000    0.0000000000    0.125
          0.5000000000   -0.50000000    0.0000000000    0.125
         -0.5000000000    0.50000000    0.0000000000    0.125
         -0.5000000000   -0.50000000    0.0000000000    0.125
          0.5000000000    0.50000000    0.5000000000    0.125
          0.5000000000   -0.50000000    0.5000000000    0.125
         -0.5000000000    0.50000000    0.5000000000    0.125
         -0.5000000000   -0.50000000    0.5000000000    0.125
      %endblock kpoints_list
   
   where k-points are defined in the fractional coordinates and the last column
   is the weight of the k-point.


Kpar parallelisation
====================

Since the calculations with k-points are almost fully isolated with one and 
another, it makes sense to use k-pool parallisation where multiple instances of 
ONETEP (k-parallisation groups, or kpars in short) is launched altogether, each
in charge of running a sub-set of k-points (and in each kpar, k-points are 
looped over in serial).

The kpar parallelisation is controlled by the keyword `num_kpar` in the input
file:

::

   num_kpars : 4

This means that the k-points are divided into 4 groups and 4 ONETEP instances
will be launched. 

It is worth noting that the number of processes needs to be the same for all
kpars. This means you need to carefully calculate the number of processes,
taking into account that each ONETEP instance can only use the number of
processes less or equal to the number of atoms in the system.


Hybrid and extended NGWFs
=========================

In the PW mode, NGWFs needs to be extended. This is turned on by using the 
keyword ``extended_ngwf`` in the input file. e.g.,

::

   extended_ngwf : T T T

It is also possible to only allow NGWFs to be extended along certain directions,
hence utlising the periodicity of the system **only** along those directions via
k-point sampling. 

It is worth nothing that the extended NGWFs can also run withotu k-point
sampling and should produce the same results as running a plan-wave DFT
calculation with only one k-point (i.e., the :math:`\Gamma` point).

Fixed kernel calculation
========================

When using fully extended NGWFs (``extended_ngwf : T T T``) it is possible to 
perform fixed kernel calculation where the number of NGWFs used is the same as 
the number of **occupied** states. Since we are not optimising the density 
kernel, this will only work with known insulators (i.e., all NGWFs are fully 
occupied).

This is done by setting:

::

   maxit_lnv=-1
   minit_lnv=-1

One thing to note is that to use this feature, sometimes one has to turn on
``permit_unusual_ngwf_count`` in the input file and play around with the number
of NGWFs for each species so that the total number of NGWFs is equal to the
number of occupied states.

Tight-binding (TB) mode
=======================

The tight-binding (TB) mode is not yet implemented. The TB mode is designed to
use fully localised NGWFs and the k-point sampling is performed by augmenting
the Hamiltonian matrix and the overlap matrix with the k-point phase factors.

Additional notes
================

Currently, full Brillouin zone sampling is only implemented in the plane-wave
mode and tested for norm-conserving pseudopotentials. 

Supported functionalities:
- Ground state energy calculation with LNV (``exact_lnv : T``) and EDFT (``edft : T``). 
- Geometry optimisation (but no cell-optimisation).
- Parts of the properties module (e.g., charge density outputs).

Keywords
========

-  ``extended_ngwf`` [Basic, bool bool bool, default ``F F F``\ ]. Turn on extended NGWFs along the three directions.

-  ``kpoint_method`` [Basic, default ``None``\ ]. The method used to generate the k-point grid. The options are:

   -  ``PW``: Plane-wave mode.
   -  ``None``: No k-point sampling.
   -  ``TB``: Currently not implemented.

- ``kpoint_grid_shift`` [Basic int int int, default ``0 0 0``\ ]. The shift of the k-point grid.

- ``kpoint_grid_size`` [Basic int int int, default ``1 1 1``\ ]. The size of the k-point grid.

- ``num_kpars`` [Basic int, default ``1``\ ]. The number of k-parallelisation groups.
