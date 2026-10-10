.. _bandstructure_pw:

=====================================================
Bandstructures with k-points (PW mode)
=====================================================

:Author: Nicholas Hine, University of Warwick, assisted by Claude Opus 5.5
:Date: October 2026

Overview
========

After a ground-state calculation with k-point sampling in plane-wave (PW)
mode (see :doc:`kpoints_and_spin`), ONETEP can calculate the bandstructure
along a path through the Brillouin zone by optimising a set of NGWFs at
each k-point on the path. This gives bands that are accurate both below
and above the Fermi level, unlike the eigenvalues of the ground-state
NGWFs, which describe only the occupied states (and a few states above
them) well.

The calculation is non-self-consistent: the density, and hence the local
potential, the PAW nonlocal terms and any Hubbard terms, are those of the
ground state, and are kept fixed. At each path k-point, one set of NGWFs
is optimised to represent the lowest :math:`N_b` bands, using the same
machinery as the conduction NGWF optimisation (see :doc:`conduction`),
but without projecting out the valence states: the bandstructure NGWFs
describe all the bands, from the lowest upwards, in a single basis. The
initial NGWFs at each k-point are Bloch sums of pseudoatomic orbitals
(PAOs) at that k-point.

The path k-points are independent of each other, so they are divided
between the k-point parallelisation groups (``num_kpars``) of the
calculation.

Tasks
=====

Two tasks perform the calculation:

-  ``BANDSTRUCTURE`` optimises the bandstructure NGWFs, then writes the
   bands and, optionally, their projections. It can follow a ground-state
   calculation in the same run, for example
   ``task : SINGLEPOINT BANDSTRUCTURE``, or read the ground-state NGWFs and
   density kernel from a previous run (``read_tightbox_ngwfs : T`` and
   ``read_denskern : T``).

-  ``PROPERTIES_BANDSTRUCTURE`` reads the bandstructure NGWFs and density
   kernel written by a previous ``BANDSTRUCTURE`` run, and only writes the
   bands and projections. It is a quick way to calculate projections of a
   different type, or to repeat the output, without optimising the NGWFs
   again. The path must be the same as in the run that wrote the files;
   this is checked (see `Output files`_).

Both tasks require ``kpoint_method : PW``. Without it, they are replaced by
``PROPERTIES``, which calculates the bandstructure in the ground-state NGWF
basis (see ``bs_method`` and :ref:`bsunfold` for those calculations).

Input
=====

The following are required, in addition to the input for the ground-state
calculation:

-  ``%block species_cond``, defining the bandstructure NGWFs of each
   species, in the same format as ``%block species``. There must be at
   least as many bandstructure NGWFs as bands to be calculated, and a few
   more is better. The PAOs used as the initial NGWFs, and as the
   projections for ``bs_projections : PAO_LOWDIN`` or ``PAO_MULLIKEN``,
   are those of this block.

-  ``%block bs_kpoint_path``, the vertices of the path in fractional
   coordinates of the reciprocal lattice vectors, and optionally
   ``bs_kpoint_path_spacing``, the spacing of the k-points along it.

-  ``extend_ngwf : T`` along every direction in which the path moves. For
   a slab with ``extend_ngwf : T T F``, for example, all the vertices of
   the path must have a zero third component.

-  A choice of the number of bands, with ``cond_num_states`` or
   ``cond_energy_range`` (see `Number of bands`_).

The NGWFs are complex at the path k-points, so ``use_cmplx_ngwfs`` is set
automatically.

The NGWF optimisation is controlled by the same keywords as the
ground-state NGWF optimisation (``maxit_ngwf_cg``, ``ngwf_threshold_orig``
and so on), and the density kernel optimisation by ``cond_maxit_lnv``, as
for conduction calculations (see :doc:`conduction`).
``cond_num_extra_its`` gives a number of NGWF iterations to perform before
the density kernel is re-initialised, which can help when the initial PAOs
are a poor guess for some bands.

An example, after a ground-state calculation of bulk silicon (two atoms
in the primitive face-centred cubic cell) with k-points, for the four
valence bands, which lie below the band gap along the whole path::

    task : BANDSTRUCTURE
    kpoint_method : PW
    extend_ngwf : T T T
    read_tightbox_ngwfs : T
    read_denskern : T

    cond_num_states : 8
    bs_projections : PAO_LOWDIN
    bs_kpoint_path_spacing : 0.05 1/bohr

    %block species_cond
    Si Si 14 9 8.0
    %endblock species_cond

    %block bs_kpoint_path
    0.0 0.0 0.0
    0.5 0.0 0.5
    0.5 0.25 0.75
    0.0 0.0 0.0
    %endblock bs_kpoint_path

QC test ``test135`` contains two complete examples, one of them with
k-point parallelisation.

Number of bands
===============

The density kernel of the bandstructure NGWFs represents the lowest
:math:`N_b` bands at each path k-point, and the NGWFs are optimised to
minimise the sum of their energies. The choice of :math:`N_b` matters:

-  If the highest band, :math:`N_b`, is degenerate with band
   :math:`N_b+1` at some k-point (for example because of symmetry), the
   lowest :math:`N_b` bands are not well defined there, the density kernel
   cannot be made idempotent, and the NGWF optimisation stalls.

-  If there is a gap above band :math:`N_b` along the whole path, the
   optimisation converges well, and all :math:`N_b` bands are accurate.

-  Bands near the top of the set, where it does not end at a gap, are less
   accurate than the bands below them.

The best choice is therefore to put the top band below a gap that persists
along the whole path, if there is one, and otherwise to choose
:math:`N_b` to include a margin of bands above the highest band of
interest.

There are two ways to set :math:`N_b`:

``cond_num_states``
   The number of bands, counting the bands of both spins: without spin
   polarisation it must be even, and gives :math:`N_b` =
   ``cond_num_states``/2. With spin polarisation, as for conduction
   calculations, spin up has (``cond_num_states`` :math:`-` ``spin``)/2
   bands and spin down (``cond_num_states`` :math:`+` ``spin``)/2 bands.

``cond_energy_range`` (with ``cond_num_states : 0``, the default)
   An energy window above a reference energy: the eigenvalues of the
   ground-state Hamiltonian are calculated at the k-points of the
   ground-state calculation and, for each spin, :math:`N_b` is the largest
   number of them below the reference plus ``cond_energy_range`` at any
   k-point. The reference is

   -  the highest occupied eigenvalue, if every spin has a gap above its
      occupied states at all k-points (an insulator or semiconductor);

   -  otherwise, the Fermi energy of the ground-state eigenvalues, with
      the smearing width ``edft_smearing_width``, common to both spins.

   With spin polarisation, the window therefore ends at the same energy in
   both spins, and the two spins can have different numbers of bands.

   :math:`N_b` is then lowered, if necessary, until the top band of each
   spin lies below a gap of at least :math:`10^{-5}` Ha at every k-point of
   the ground-state calculation, so that it does not split a set of
   degenerate states. If ``cond_num_extra_its`` is greater than zero, the
   check is repeated after those iterations, starting again from the
   number of bands in the window, with the eigenvalues of the bandstructure
   NGWFs at the path k-points.

   The window mode is recommended, and in particular for spin-polarised
   systems.

The output reports the reference energy, the number of bands in the window
for each spin, any reduction to avoid degenerate states, and, at the end,
the smallest gap above the top band along the path for each spin, with a
warning if it is zero.

The ground-state eigenvalues used for the window are calculated in the
ground-state NGWF basis, which describes the states above the Fermi level
less well than the bandstructure NGWFs will, and at the k-points of the
ground-state calculation rather than those of the path. The choice of
:math:`N_b` is therefore not guaranteed to avoid degeneracies along the
path, although the repeated check after ``cond_num_extra_its`` makes this
more likely.

**Metals.** In a metal there is often no gap above the bands of interest
along the whole path. The window mode still gives a usable set of bands,
but the NGWF optimisation may converge slowly. For example, for
ferromagnetic bcc Fe with ``cond_energy_range : 3.0 eV``, the window
contains 20 bands in each spin, which splits a pair of degenerate bands at
:math:`\Gamma` and along :math:`\Gamma`-R; with the 19 bands per spin
chosen after the degeneracy check, the NGWF optimisation proceeds
normally and the bands agree with a plane-wave calculation to within the
difference between the two ground-state calculations, although the NGWF
gradient decreases slowly in the later iterations.

Band projections
================

``bs_projections`` gives the weight of each atom or each PAO in each band,
at each path k-point. With :math:`M` the eigenvectors of the Hamiltonian in
the bandstructure NGWF basis (in columns), :math:`S` the overlap matrix of
the NGWFs, :math:`O` the overlap matrix of the PAOs with the NGWFs, and
:math:`\Lambda` the overlap matrix of the PAOs, the weights of band
:math:`n` are:

``ATOM_MULLIKEN``
   :math:`w_{An} = \sum_{\alpha \in A} \mathrm{Re}\left[ M^*_{\alpha n} (S M)_{\alpha n} \right]`,
   the Mulliken population of the NGWFs on atom :math:`A`.

``ATOM_LOWDIN``
   :math:`w_{An} = \sum_{\alpha \in A} \left| (S^{1/2} M)_{\alpha n} \right|^2`,
   the Löwdin population of the NGWFs on atom :math:`A`.

``PAO_LOWDIN``
   :math:`w_{\mu n} = \left| (\Lambda^{-1/2} O M)_{\mu n} \right|^2`, for
   each PAO :math:`\mu`.

``PAO_MULLIKEN``
   :math:`w_{\mu n} = \mathrm{Re}\left[ (\Lambda^{-1} O M)^*_{\mu n} (O M)_{\mu n} \right]`.

``NONE`` (the default) gives no projections. All the overlaps include the
PAW augmentation terms when PAW is used. For the PAO projections, the
PAOs are those of ``%block species_cond``, constructed at each path
k-point.

For the ``ATOM`` modes, the weights of each band sum to 1. For the ``PAO``
modes, the sum is less than 1 by the *spilling* of the band, the part of it
that the PAOs do not represent, which is written with the weights and
should be small (typically below 0.02). The Löwdin weights are
non-negative; the Mulliken weights can be slightly negative.

The PAO weights are given for each shell and each real spherical harmonic.
Diffuse PAO shells (for example an unoccupied d shell of a p-block
element) overlap strongly with the orbitals of neighbouring atoms, and can
take weight that would more naturally be assigned to those orbitals: the
weights of such shells should be interpreted with care.

Spin-orbit coupling
===================

With ``bs_perturbative_soc : T``, spin-orbit coupling (SOC) is added to the
bands at each path k-point. The NGWFs, and the ground state, are calculated
without SOC, in the scalar-relativistic approximation and with collinear
spin; SOC is included only when the bands are written, as for the
perturbative SOC of ``PROPERTIES`` bandstructure calculations. This is a
second-variation approach: SOC mixes the states within the span of the
bandstructure NGWFs.

At each path k-point, the basis is the bandstructure NGWFs
:math:`\{\phi_\alpha\}` times the two spin states, giving :math:`2N` spinor
basis functions for :math:`N` NGWFs. The spinor Hamiltonian and overlap are

.. math::

   H = \begin{pmatrix} H_0^{\uparrow} + \Delta^{\uparrow\uparrow} & \Delta^{\uparrow\downarrow} \\
       \Delta^{\downarrow\uparrow} & H_0^{\downarrow} + \Delta^{\downarrow\downarrow} \end{pmatrix},
   \qquad
   S_{\mathrm{so}} = \begin{pmatrix} S & 0 \\ 0 & S \end{pmatrix},

where :math:`H_0^{\sigma}` is the Hamiltonian of the bandstructure NGWFs
without SOC (the same for both spins without spin polarisation), :math:`S`
their overlap matrix, and

.. math::

   \Delta^{\sigma\sigma'}_{\alpha\beta} = \sum_{ij} \langle \phi_\alpha | p_i \rangle
   D^{\mathrm{so},\sigma\sigma'}_{ij} \langle p_j | \phi_\beta \rangle

is the SOC term, from the projectors :math:`p_i` and on-site SOC
coefficients :math:`D^{\mathrm{so}}`. :math:`D^{\mathrm{so}}` couples only
projectors on the same atom with the same angular momentum :math:`l \neq
0`, and is written below for projectors :math:`i = (n, l, m_i)` and
:math:`j = (n', l, m_j)` of one atom, with :math:`m` labelling the real
spherical harmonics :math:`S_{lm}` of the projectors. Both forms involve
the matrix elements of :math:`\mathbf{L} \cdot \boldsymbol{\sigma}` in the
basis :math:`S_{lm} \chi_\sigma`, which in spin blocks are

.. math::

   \langle l m_i \sigma | \mathbf{L} \cdot \boldsymbol{\sigma} | l m_j \sigma' \rangle =
   \begin{pmatrix}
   \langle S_{l m_i} | L_z | S_{l m_j} \rangle & \langle S_{l m_i} | L_- | S_{l m_j} \rangle \\
   \langle S_{l m_i} | L_+ | S_{l m_j} \rangle & -\langle S_{l m_i} | L_z | S_{l m_j} \rangle
   \end{pmatrix}_{\sigma\sigma'} .

**PAW.** :math:`D^{\mathrm{so}}` is calculated from the all-electron partial
waves :math:`\phi_{nl}(r)` (radial parts) and the spherical part
:math:`V(r)` of the all-electron potential inside the augmentation sphere
of radius :math:`r_c`:

.. math::

   D^{\mathrm{so},\sigma\sigma'}_{ij} = \frac{\alpha^2}{4}
   \int_0^{r_c} \phi_{nl}(r) \, \phi_{n'l}(r) \,
   \frac{1}{1 - \alpha^2 V(r)} \frac{1}{r} \frac{dV}{dr} \, dr \;
   \langle l m_i \sigma | \mathbf{L} \cdot \boldsymbol{\sigma} | l m_j \sigma' \rangle ,

in atomic units, with :math:`\alpha` the fine-structure constant (so that
:math:`\alpha^2/4 \; (1/r)(dV/dr) \, \mathbf{L} \cdot \boldsymbol{\sigma}`
is the spin-orbit operator :math:`(1/2c^2)(1/r)(dV/dr) \, \mathbf{L} \cdot
\mathbf{S}`, with :math:`\mathbf{S} = \boldsymbol{\sigma}/2`), and
:math:`1/(1 - \alpha^2 V)` a relativistic (ZORA-type) correction near the
nucleus. :math:`V` is the potential of the nucleus, the Hartree potential
of the on-site all-electron valence density (from the density matrix
:math:`\rho_{ij}` of the ground state) and the core density, and the
exchange-correlation potential; with spin polarisation, it is averaged
over the two spins.

**Norm-conserving pseudopotentials.** The pseudopotentials must be fully
relativistic: UPF files with projectors for total angular momentum
:math:`J = l \pm 1/2`, with coefficients :math:`D_{nlJ}`. In the basis of
real spherical harmonics and spins, the projectors onto the two values of
:math:`J` are

.. math::

   P_{l+1/2} = \frac{l + 1 + \mathbf{L} \cdot \boldsymbol{\sigma}}{2l + 1}, \qquad
   P_{l-1/2} = \frac{l - \mathbf{L} \cdot \boldsymbol{\sigma}}{2l + 1},

and

.. math::

   D^{\mathrm{so},\sigma\sigma'}_{ij} = \delta_{nn'} \left(
   \sum_{J = l \pm 1/2} D_{nlJ} \, \langle l m_i \sigma | P_J | l m_j \sigma' \rangle
   - D^{\mathrm{SR}}_{nl} \, \delta_{m_i m_j} \delta_{\sigma\sigma'} \right),

where :math:`D^{\mathrm{SR}}_{nl}` is the scalar-relativistic coefficient
already included in :math:`H_0`.

:math:`D^{\mathrm{so}}` is independent of k; the projector overlaps
:math:`\langle \phi_\alpha | p_i \rangle` are those at the path k-point.

The generalised eigenvalue problem gives :math:`2N` spinor bands at each
k-point, of which :math:`2N_b` are written, with :math:`N_b` the number of
bands of each spin chosen as in `Number of bands`_ (with spin
polarisation, the sum of the numbers of bands of the two spins). The number
of bands, and the check for degenerate states at the top of the set, refer
to the bands without SOC, which are the bands that the NGWFs are optimised
for.

**Accuracy.** The SOC is exact within the space spanned by the NGWFs. The
bands are accurate where SOC mainly mixes bands within the set and the
NGWFs describe them well; bands near the top of the set, which SOC mixes
with bands above it, are less accurate. The approach is suited to systems
in which SOC is a perturbation on the scalar-relativistic bands; the
density and the NGWFs do not respond to SOC.

**Spin polarisation.** With spin polarisation, the up and down Hamiltonians
differ, and the magnetisation is along the :math:`z` axis of the cell. This
case is less well tested than the unpolarised one, and a warning is
printed.

**Projections.** For an eigenvector :math:`c = (c^{\uparrow}, c^{\downarrow})`,
the overlaps :math:`S`, :math:`O` and :math:`\Lambda` do not couple the two
spins, so each of the formulas of `Band projections`_ gives the sum of its
values for :math:`c^{\uparrow}` and :math:`c^{\downarrow}`. These sums are
written as the band weights. The expectation value of the spin along
:math:`z`,

.. math::

   \langle S_z \rangle = \frac{1}{2} \left( c^{\uparrow\dagger} S c^{\uparrow}
   - c^{\downarrow\dagger} S c^{\downarrow} \right),

is written as the last column of ``<root>_BS.pdos_weights``.

In a system with inversion symmetry, every band is doubly degenerate at
every k-point (Kramers degeneracy), as it is at the time-reversal-invariant
k-points of any system. Within a degenerate pair, the split of the states
is arbitrary: :math:`\langle S_z \rangle` of an individual band is not
meaningful, and nor are the weights of individual atoms that inversion
maps onto each other, although their sums over such atoms are. Without
inversion symmetry, as in a monolayer of 2H-MoS\ :sub:`2`, the bands away
from these k-points are not degenerate, and :math:`\langle S_z \rangle`
gives their spin polarisation.

**Example.** SOC can be added to bandstructure NGWFs optimised in an
earlier ``BANDSTRUCTURE`` run without SOC, with ``PROPERTIES_BANDSTRUCTURE``,
which reads the NGWFs and only calculates the bands::

    task : PROPERTIES_BANDSTRUCTURE
    bs_perturbative_soc : T
    bs_projections : ATOM_MULLIKEN

with the rest of the input as for the ``BANDSTRUCTURE`` run. Equally,
``bs_perturbative_soc : T`` can be set in the ``BANDSTRUCTURE`` run itself.
For example, for a monolayer of 2H-MoS\ :sub:`2` with the 13 valence bands
(``cond_num_states : 26``), the top valence band at K is split by about
0.15 eV into two bands with :math:`\langle S_z \rangle \approx \pm 1/2`.

Output files
============

``<root>_BS.bands``
   The eigenvalues of the bandstructure NGWFs at each k-point on the path,
   in the format of the ``.bands`` files of ``PROPERTIES`` bandstructure
   calculations (and CASTEP). The Fermi energy is the highest occupied
   eigenvalue along the path or, for a metal (see `Number of bands`_), the
   Fermi energy of the ground state. With ``bs_perturbative_soc : T``, there
   is a single set of :math:`2N_b` spinor bands (one independent spin
   component).

``<root>_BS.pdos_weights``
   With ``bs_projections``, the band weights. The header gives the
   projection type, the numbers of k-points, spins, bands and
   projections, the number of bands of each spin, the reference energy
   (in Ha, see `Number of bands`_), and a table describing each
   projection: its atom and species and, for the PAO modes, its shell,
   angular momentum :math:`l`, :math:`m` and orbital name (for example
   ``px`` or ``dxy``). Then, for each k-point and spin, a line
   ``# K-point`` with the index, the fractional coordinates of the k-point
   and the spin, followed by one line for each band, with the band index,
   the energy (Ha), the spilling and the weights. With
   ``bs_perturbative_soc : T``, a header line marks the file as containing
   spinor bands, and each band line ends with :math:`\langle S_z \rangle`
   (see `Spin-orbit coupling`_).

``<root>_BS.agr``
   With ``bs_write_agr : T``, the bands in xmgrace format.

``<root>.tightbox_ngwfs_bs``, ``<root>.dkn_bs``
   The bandstructure NGWFs and density kernel, for
   ``PROPERTIES_BANDSTRUCTURE``. They are written with
   ``write_tightbox_ngwfs : T`` and ``write_denskern : T``.

``<root>.kpoints_bs``
   The path k-points that the NGWFs in the ``_bs`` files belong to. When
   these files are read, the path is checked against this record.

With k-point parallelisation, the ``_bs`` files and the ``.kpoints_bs``
file are written by each k-point group, with the group index appended to
the root name, as for the ground-state files.

Restrictions
============

-  Hybrid functionals are not supported.

-  Spin-orbit coupling is perturbative only: it is not included in the
   ground state or in the NGWF optimisation (see `Spin-orbit coupling`_).

-  ``edft`` is set to F, and ``ngwf_cg_type : NGWF_LBFGS`` is replaced by
   ``NGWF_FLETCHER``, for these tasks, with a warning: the bandstructure
   NGWFs are optimised for a fixed number of bands at each k-point. The
   ground-state calculation in the same run can use EDFT.

-  The density kernel represents a fixed number of bands at each path
   k-point. For metals with no gap above the bands of interest, see
   `Number of bands`_.

Keywords
========

-  ``task`` [Basic, string] ``BANDSTRUCTURE`` or
   ``PROPERTIES_BANDSTRUCTURE``.

-  ``bs_kpoint_path`` [Basic, block] The vertices of the path, in
   fractional coordinates of the reciprocal lattice vectors.

-  ``bs_kpoint_path_spacing`` [Intermediate, physical, default
   ``0.1889727 1/bohr``] The spacing of the k-points along the path.

-  ``species_cond`` [Basic, block] The bandstructure NGWFs of each
   species.

-  ``cond_num_states`` [Basic, integer, default ``0``] The number of
   bands, counting both spins. With ``0``, ``cond_energy_range`` is used.

-  ``cond_energy_range`` [Intermediate, physical, default ``-1.0 Ha``] The
   energy window for the number of bands, above the highest occupied
   eigenvalue or the Fermi energy.

-  ``cond_num_extra_its`` [Intermediate, integer, default ``0``] NGWF
   iterations before the density kernel is re-initialised and the number of
   bands is checked again.

-  ``bs_projections`` [Intermediate, string, default ``NONE``] The band
   weights: ``NONE``, ``ATOM_MULLIKEN``, ``ATOM_LOWDIN``, ``PAO_LOWDIN`` or
   ``PAO_MULLIKEN``.

-  ``bs_perturbative_soc`` [Basic, logical, default ``F``] Add spin-orbit
   coupling perturbatively to the bands (see `Spin-orbit coupling`_).

-  ``bs_write_agr`` [Intermediate, logical, default ``F``] Write the bands
   in xmgrace format.

-  ``num_kpars`` [Basic, integer, default ``1``] The number of k-point
   parallelisation groups, between which the path k-points are divided.
