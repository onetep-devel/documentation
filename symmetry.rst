==========================
Crystal symmetry in ONETEP
==========================

:Author: Chengcheng Xiao
:Date:   July 2025

Symmetry operations in ONETEP
=============================

Crystal symmetry operations can be defined using rotation and translation pairs.
In ONETEP, symmetry operations (in fractional coordinates) are obtained by
passing the crystal information (lattice, ionic positions, and types) to
`spglib <http://github.com/spglib/spglib>`.


These operations can then be used to find symmetry-related k-points:

.. math::

   \mathbf{k}_\mathrm{rot} = \mathbf{R} \mathbf{k}

where :math:`\mathbf{R}` is the rotation operator in reciprocal space, which is
the inverse of the rotation operator in the real space.

Symmetry-related k-points have the same energy; hence, we can use only one
k-point for each k-point group that is symmetry-related to achieve more
efficient calculations. Note that the k-point weights are also adjusted based on
the number of symmetry-related k-points in the group.

In addition to the crystal symmetry, ONETEP also supports time-reversal symmetry
(TRS) to further reduce the number of k-points. In practice, this means that one
can eliminate half of the k-points in the first Brillouin zone because TRS
dictates that k-points (k) and their time-reversed counterparts (-k) have the
same energy.

However, while the energy calculated by symmetry-related k-points is the same,
the wavefunctions are not. Luckily, the wavefunctions of symmetry-related
k-points are also symmetry-related, and we symmetrise the charge density (for
each spin channel if spin-polarised) so that the charge density and energy is
correct.

The symmetrisation of the charge density is done in reciprocal space. First, the
charge density in the simulation cell is Fourier transformed to reciprocal
space. Then, we apply the rotation operations in fractional coordinates to the
indices of the G-vectors to find the indices of their symmetry-related
counterparts,

.. math::

   \mathbf{G}_\mathrm{rot} = \mathbf{R} \mathbf{G}

where :math:`\mathbf{G}` is the G-vector in reciprocal space. Finally, we
extract the corresponding value of the charge density at the symmetry-related
point in reciprocal space and perform a phase-weighted average. The phase is
caused by the non-primitive cell translation part of the symmetry operation. And
the whole process is repeated for each symmetry operation. Mathematically this
can be expressed as:

.. math::

   n(\mathbf{G}) = \frac{1}{N} \sum_i^N n( \textbf{R}_i \mathbf{G})e^{i2 \pi
   \textbf{R}_i \mathbf{G}\cdot T_i}

We also leverage the fact that if the cell we are calculating is not primitive,
we'll have pure non-primitive translation operations, and if a G-vector is not
commensurate with the total number of non-primitive translations, it will have
zero amplitude so we can **avoid** applying symmetry operations to these
G-vectors altogether.

Magnetic symmetry operations
============================

Magnetic moments can affect the symmetry operations in a crystal. Specifically,
the presence of certain magnetic moments can lead to the existence of additional
symmetry operations - Time-reversal operations that swap the spin components of
the wavefunctions.

In ONETEP, instead of using full time-reversal operations to symmetrise the spin
polarised charge density, the entire crystal symmetry is lowered by treating the
atoms that belong to the same species but have different **initial** magnetic
moments as if they belong to different species.

Doing this allows the possibility for the lower symmetry magnetic structure
(such as antiferromagnetic) to relax into a higher symmetry structure (such as
ferrimagnetic).


This is done automatically by ONETEP when the ``use_symmetry`` keyword is set to
``True`` and the ``use_time_reversal`` keyword is set to ``True`` and ``LOCK
SPECIES_ATOMIC_SET`` is used to specify different initial magnetic moments for
different atoms of the same species (for more, see
:ref:`initial-guess-density-setting-initial-charges-and-spins`). If the initial
magnetic moments do change the symmetry, ONETEP will report the symmetry
operations written:

.. code::

    Number of symmetry operations =          NUMBER
     (reduced by initial magmoms)


Compiling ONETEP with spglib
============================

ONETEP now ships with spglib and can be enabled by ``BUILD_SPGLIB=yes``, i.e.,

.. code::

   make onetep ARCH=YOUR_ARCH BUILD_SPGLIB=yes

You can also change the C-compiler by setting the ``CC`` environment variable in
your ARCH file, e.g.,

.. code::

   CC=gcc-11

Running ONETEP with symmetry
============================

To run ONETEP with symmetry, the user only needs to set the ``use_symmetry``
keyword to ``True``. To leverage the time-reversal symmetry to reduce the number
of k-points, the ``use_time_reversal`` keyword can also be set to `True`.

One thing to note is that only MP grids set up by ``kpoint_grid_size`` will get
symmetrised. User-supplied k-point list (via ``block kpoints_list``) will not be
modified.

Once these tags are set, ONETEP will report the symmetry operations and the
reduced k-points in the output file (if ``output_detail`` is set to
``verbose``), and the symmetry-related k-points will be used in the calculation.

Keywords
========

- ``use_symmetry``: [bool] turn symmetry on or off (only works if compiled with
  spglib) | default: off

- ``use_time_reversal`` : [bool] use TRS to reduce the number of k-points or not |
  default: on

- ``symmetry_tol``: [float] Precision in determining the symmetry. | default:
  0.001889726 Bohr (0.001 Å)

