.. _bsunfold:

=====================================================
Bandstructure (spectral-function) unfolding
=====================================================

:Author: Gabriel Constantinescu, University of Cambridge
:Author: Nicholas Hine, University of Warwick, assisted by Claude Opus 5.5

| As described in the Supplementary Information in the work of
  Constantinescu and Hine [Constantinescu2015]_,
  spectral function unfolding is the means through which one can study
  the bandstructure of the primitive cell from simulations involving
  (often complicated) supercells. For details theoretical descriptions,
  one should visit the aforementioned article
  [Constantinescu2015]_.
| In this documentation, I will give a short explanation of the
  procedures required to unfold the bandstructure in a ONETEP
  “Properties” calculation, and, from the section `Conduction states`_
  onwards, how to unfold the states of conduction NGWFs and how to use
  k-points. We strongly suggest that one reads the
  entire document carefully before attempting any calculations, as there
  are some stringent requirements along the way. Essentially,
  all one will need is the following group of keywords and blocks:

::

    BSUNFLD_CALCULATE : T
    BSUNFLD_TRANSFORMATION : t11 t12 t13 t21 t22 t23 t31 t32 t33

    %block SPECIES_BSUNFLD_GROUPS
    Species_name-1 Species_name-2 ... Species_name-N 
    %endblock SPECIES_BSUNFLD_GROUPS

    BS_PERTURBATIVE_SOC : F
    BSUNFLD_NUM_ATOMS_PRIM : nat_prim
    BSUNFLD_RESTART : F
    BSUNFLD_NUM_EIGENVALUES : num_eigvl

    %block BSUNFLD_KPOINT_PATH
     fraction_G1_kpt-1 fraction_G2_kpt-1 fraction_G3_kpt-1
     fraction_G1_kpt-2 fraction_G2_kpt-2 fraction_G3_kpt-2
     ...
     fraction_G1_kpt-N fraction_G2_kpt-N fraction_G3_kpt-N
    %endblock BSUNFLD_KPOINT_PATH

    BSUNFLD_NUM_KPTS_PATH : num_kpts_path

For each in turn:

-  | ``BSUNFLD_CALCULATE`` setting this to True will enable the calculation of the unfolded
     spectral function. Default: False

-  | ``BSUNFLD_TRANSFORMATION`` is a list of 9 integers representing the flattened transformation
     matrix from the primitive cell to the supercell: :math:`\textbf{T}= \begin{bmatrix} t11 & t12 & t13 \\ t21 & t22 & t23 \\ t31 & t32 & t33 \end{bmatrix}`, where :math:`\textbf{T} \begin{bmatrix} \mathbf{a_1} \\ \mathbf{a_2} \\ \mathbf{a_3} \end{bmatrix} = \begin{bmatrix} \mathbf{A_1} \\ \mathbf{A_2} \\ \mathbf{A_3} \end{bmatrix}`, with :math:`\mathbf{a_i}` (:math:`\mathbf{A_i}`) being the
     lattice vectors of the primitive cell (supercell). For instance, ``BSUNFLD_TRANSFORMATION`` :
     4 0 0 0 4 0 0 0 1 would correspond to an implicit 4x4x1 supercell.
     Default: 1 0 0 0 1 0 0 0 1, corresponding to a 1x1x1 supercell.

-  | The block ``SPECIES_BSUNFLD_GROUPS`` contains the atoms on which we are projecting; one needs to
     specify the of the atoms species (can be different from the chemical
     symbol) on which the spectral function is projected. Currently, only
     one group at a time is allowed. Example:

     ::

         %block SPECIES_BSUNFLD_GROUPS
         Mo Se1
         %endblock SPECIES_BSUNFLD_GROUPS

     this projects onto the MoSe\ :math:`_2` monolayer in a
     MoSe\ :math:`_2`/WSe\ :math:`_2` heterostructure, where the Se atoms
     belonging to the MoSe\ :math:`_2` monolayer have been denoted as
     *Se1*.

-  | The logical keyword ``BS_PERTURBATIVE_SOC`` controls the inclusion (True) or exclusion
     (False) of perturbative spin-orbit coupling in our calculation.
     Note that if set to True, the eigenvalues are no longer
     spin-degenerate and one will have twice the number of states. Since
     this option essentially quadruples the size of the Hamiltonian and
     overlap matrix, the calculation will be significantly slower.
     Default: False.

-  | The integer keyword ``BSUNFLD_NUM_ATOMS_PRIM`` indicates the number of atoms in the primitive
     cell onto which you are unfolding. This brings us to a major
     requirement of the code: while the projected atoms can be split
     across the list of all input atoms, **the former must be grouped by primitive cells, always maintaining the same order of the atoms in the primitive cells!** Example: if atoms **At1** and **At2** form a
     primitive cell, the input ordering “At3 **At1 At2** At4 **At1 At2** A3” is correct, while
     “At3 **At1 At2** At4 **At2** At3 **At1**” or “At3 **At1 At2** At4 **At2 At1** At3” are incorrect.

-  | The logical keyword ``BSUNFLD_RESTART`` controls whether one wishes to reuse previously
     calculated values for the spectral function. The restart procedure
     works as follows: the code writes a restart file
     (“unfolded\ :math:`\_`\ specfunc\ :math:`\_`\ red.dat”) where it
     stores the calculated values for the unfolded spectral function at
     unique k-points of the monolayer - the values that have not been
     calculated yet are filled with zeros. If this file is present and the
     restart keyword is set to true, the code will read in the previously
     calculated values and skip them in the current computations. The
     final output file (“unfolded\ :math:`\_`\ specfunc.dat”) will only be
     written once all the k-points have been dealt with. The files are
     ``<seedname>_unfolded_specfunc_red.txt`` and
     ``<seedname>_unfolded_specfunc.txt``, with ``_G<n>`` before ``.txt``
     for the groups of ``SPECIES_BSUNFLD_GROUPS`` after the first. Restarting
     is not available with ``NUM_KPARS`` greater than 1.

   | Each line in final output file or the restart file contains 8
     numbers: the eigenvalue (in eV), the real part of the unfolded
     spectral function (at that k-point and eigenvalue), the imaginary
     part of the unfolded spectral function, the eigenvalue count (from 1
     to :math:`2\cdot` BSUNFLD\ :math:`\_`\ NUM\ :math:`\_`\ EIGENVALUES,
     the x, y, and z component of the current primitive-cell k-point
     (:math:`\AA^{-1}`), and the index of the k-point (from 1 to the total
     number of considered k-points). Summed over all the states of the
     supercell, the spectral function at each k-point is the number of
     NGWFs in the primitive cell, which is a useful check when
     ``BSUNFLD_NUM_EIGENVALUES`` includes all the states.

   | After the final output file has been obtained, one can discretize
     the result by convolving with a gaussian with a suitably-chosen width
     (perhaps try something around 0.05eV) in order to obtain a E(k) file
     that is ready to plot with gnuplot.

-  | The integer keyword ``BSUNFLD_NUM_EIGENVALUES`` controls the number of eigenvalues (above and
     below the Fermi level) for which the spectral function is calculated.
     If set to negative values,
     ``BSUNFLD_NUM_EIGENVALUES`` will internally be
     set to the number of nondegenerate occupied eigenstates. Thus, in
     total, for each k-point, the spectral function will be calculated at
     :math:`2\cdot` ``BSUNFLD_NUM_EIGENVALUES``.

-  | The block ``BSUNFLD_KPOINT_PATH`` indicates the fractional coordinates of the different
     k-points that mark endpoints of desired paths through the **primitive-cell Brillouin zone. The fractional coordinates are with respect to the implied reciprocal lattice vectors of the primitive cell, not the simulation cell (supercell).**

-  | ``BSUNFLD_NUM_KPTS_PATH`` represents the number of k-points calculated along each path from
     the ``BSUNFLD_KPOINT_PATH`` block. This number includes the endpoints of each path. Default:
     2 (the endpoints only)

Conduction states
=================

In a conduction calculation (``TASK : COND`` or ``PROPERTIES_COND``, or
``NSCF`` or ``PROPERTIES_NSCF``), the
spectral function is that of the states of the conduction stage, and the
states of the valence NGWFs are not unfolded. With unprojected conduction
NGWFs (``COND_UNPROJECTED : T``, or a conduction calculation with EDFT; see
:ref:`cond_unprojected`), the conduction NGWFs describe all the states up to the
top of the conduction window, occupied by the valence electrons. The
occupied states are then counted from the valence electrons, and
``BSUNFLD_NUM_EIGENVALUES`` may extend up to the top of the window; by
default all the states of the window are unfolded. The energies are not
shifted: the Fermi energy or valence-band maximum is in the
``.proj_bands`` file of the conduction calculation.

k-points and extended NGWFs
===========================

Spectral function unfolding needs the supercell states at the supercell
k-point :math:`\mathbf{K}` to which each primitive-cell k-point of the path
folds. With localised NGWFs, these are obtained from the Hamiltonian and
overlap matrices of the calculation by a phase for each pair of NGWFs that
overlap through a periodic image, which requires the NGWF diameter to be
smaller than half of the lattice vector along which the phase acts.

Along lattice vectors along which the primitive cell repeats exactly, as in
a multilayer stack or a superlattice, or a defect in a slab, the supercell
need not be larger than the primitive cell: the NGWFs may be extended along
them (``EXTEND_NGWF``) and the Brillouin zone sampled with k-points
(``KPOINT_METHOD : PW``), with the supercell built only along the other
lattice vectors. Each k-point of the path is then matched to the k-point
of the calculation with the same components along the extended lattice
vectors (modulo reciprocal lattice vectors), whose matrices are used, with
the phases acting only along the other lattice vectors. If there is no such
k-point, the k-point of the calculation at :math:`-\mathbf{K}` is used,
since the spectral function of a system with time-reversal symmetry is the
same at :math:`\mathbf{k}` and :math:`-\mathbf{k}`. The calculation stops
if neither is present.

The k-points of the calculation can come from:

-  the k-point mesh of the ground state, or of a conduction calculation.
   The mesh must include the components of the path along the extended
   lattice vectors, so ``KPOINT_GAMMA_CENTRED : T`` is usually needed, and
   any segment of the path along an extended lattice vector must have
   ``BSUNFLD_NUM_KPTS_PATH`` such that its points fall on the mesh.

-  the k-point path of a conduction bandstructure calculation
   (``TASK : NSCF`` with ``NSCF_KPOINTS : PATH``, see
   :ref:`bandstructure_pw`), along the
   extended lattice vectors. The spectral function can then be unfolded
   along any path whose components along the extended lattice vectors lie
   on that path, with ``BSUNFLD_CALCULATE : T`` in the same calculation.

The requirements are:

-  ``BSUNFLD_TRANSFORMATION`` must be 1 along the extended lattice vectors
   and must not mix them with the others.

-  the NGWFs must not be extended along the lattice vectors of the
   supercell, and must satisfy the size condition along them, and the
   k-points of the calculation must have no components along them (which
   ONETEP requires in any case).

-  the calculation must not be spin-polarised, and perturbative spin-orbit
   coupling (``BS_PERTURBATIVE_SOC``) requires a single k-point.

With ``NUM_KPARS`` greater than 1, each kpar unfolds the k-points of the
path matched to its own k-points, and the output files are written once,
without the kpar index in their names.

The weights of path points that fold to a supercell k-point other than
:math:`\Gamma` were incorrect in versions before October 2026 (ONETEP
pull request 1035), which omitted the phase of the overlap of NGWFs across
the supercell boundary from the projection, mostly affecting the weights
of the higher conduction states. The energies were not affected.

.. [Constantinescu2015] Constantinescu, G. C.; Hine, N. D. M. *Phys. Rev. B* **2015**, *91*, 195416
