==============================================
Electron Energy Loss Calculations
==============================================

:Author: Edward Tait, University of Cambridge
:Author: Nicholas Hine, University of Warwick

:Date:   September 2015
:Date:	 Updated and Converted by NDMH September 2022

Theory
======

Computation of Electron Energy Loss (EEL) spectra in the Kohn-Sham formal-
ism relies on the application of Fermi’s Golden Rule to compute the imaginary
part of the dielectric function,


.. math::

   \epsilon_2(\omega) = \frac{1}{\Omega} \sum\limits_{c}\sum\limits_{i} |\langle  \psi_i| \exp(i\mathbf{r}\cdot\mathbf{q})
   | \psi_c \rangle |^2 \delta (E_i - E_c - \omega)\,,

Here :math:`\omega` is the transition energy, :math:`\Omega` the unit cell
volume, the :math:`\psi_i` are (all electron) conduction band states,
the :math:`\psi_c` are core states, with respective energies :math:`E_i`
and :math:`E_c`. :math:`\mathbf{r}` is the position operator
(defined as the displacement from the nucleus
whose core electrons are being excited). The :math:`\delta`-function
conserves energy. :math:`\textsc{ONETEP}` relies on the external tool
OptaDoS[1] for the computation of :math:`\epsilon_2`
and only needs to supply matrix elements in a compatible form. The rest of
this section discusses the calculation of these elements.

Projector Augmented Wave
------------------------

In common with many plane wave codes onetep uses pseudopotentials to increase
computational efficiency. A drawback of pseudopotentials is poor
representation of the all-electron Kohn-Sham wavefunction close to the nucleus,
this however is exactly the region in which the matrix elements used for EELS
simulation are computed.
To overcome this obstacle the projector augmented wave (PAW) formalism
of Blochl[2] is adopted, which permits reconstruction of all-electron
states :math:`\psi`
(and thus matrix elements) from pseudo-wavefunctions :math:`\widetilde{\psi}`:

.. math::
   :label: paw_matel

   \langle \phi_\alpha \vert \hat{O} \vert b \rangle =
   \underbrace{ \langle \phi_\alpha \vert  \hat{O} \vert b\rangle }_\text{Cartesian Grid} +
   \sum_{i}\langle \phi_\alpha \vert \widetilde{p}_\text{i}\rangle\underbrace{(\langle \varphi_\text{i} \vert  \hat{O}
   \vert b \rangle -
   \langle \widetilde{\varphi}_\text{i}
   \vert  \hat{O} \vert b \rangle)}_\text{Radial Grids}

This process is accomplished by decomposing the pseudo-wavefunction in
the augmentation region into a sum of pseudo partial waves, :math:`\widetilde{\varphi}_i`, the weights in
this sum are determined using the projectors :math:`\widetilde{p}_i`.
These pseudo partial waves are
subtracted off and all electron partial waves, :math:`\varphi_i`, added in their place.
Simply put, within the augmentation regions (close to the nucleus) the pseudised part
of pseudo-wavefunction is subtracted off and the all electron part is added back
on.

Several PAW data sets are freely available (subject to the caveat that they
should be thoroughly tested for a specific use case). onetep accepts PAW
pseudopotentials in the abinit file format. Core wavefunctions, as produced
by the pseudopotential generator are also required, these are less frequently
available for download, but your PAW library should provide input files for a
pseudopotential generator which can be used to produce core wavefunction data
sets (again in the abinit format).

Generation of Position-Core Kets
--------------------------------

To calculate the PAW matrix elements we must compute terms of the form:
:math:`\langle\widetilde{\psi}|\mathbf{r}|\psi_\mathrm{c}\rangle`

As an intermediate we compute expressions of the form:
:math:`\langle\phi_\alpha|\mathbf{r}|\psi_\mathrm{c}\rangle`

with :math:`\phi_\alpha` an NGWF. The integral implied by this bra-ket must be computed
on the grid. The bra term, an NGWF, is readily available in this form but the
ket :math:`|\psi_\mathrm{c}\rangle` must be constructed.

A Fourier space method is used, as it was found
to offer superior numerical performance and correctly treats periodic systems
without further modification. This method exploits the fact that differentiation
in Fourier space is equivalent to multiplication by the position vector in real
space:

.. math::

   \begin{aligned}
   \psi_{\rm c}(\mathbf{r}) &= \sum\limits_\mathbf{G}^{\mathbf{G}_{\rm max}} \exp(i\mathbf{G}\cdot\mathbf{r}) \\
   \rightarrow \sum\limits_\mathbf{G}^{\mathbf{G}_{\rm max}}( \nabla_\mathbf{G}(\psi_{\rm c}(\mathbf{G}))\exp(i\mathbf{G}\cdot\mathbf{r}) &= \sum\limits_\mathbf{G}^{\mathbf{G}_{\rm max}}( \nabla_\mathbf{G}(\psi_{\rm c}(\mathbf{G})\exp(i\mathbf{G}\cdot\mathbf{r})) \\
   &-\sum\limits_\mathbf{G}^{\mathbf{G}_{\rm max}} \psi_{\rm c}(\mathbf{G}) \nabla_\mathbf{G} \exp(i\mathbf{G}\cdot\mathbf{r}) \\
   &=-\sum\limits_\mathbf{G}^{\mathbf{G}_{\rm max}} i \mathbf{r} \psi_{\rm c}(\mathbf{G}) \exp(i\mathbf{G}\cdot\mathbf{r})) \\
   &=-i\mathbf{r}  \psi_{\rm c}(\mathbf{r})
   \end{aligned}

Conduction Optimisation
-----------------------

Core loss calculations rely on an accurate description of conduction band states,
in onetep it is necessary to perform a conduction optimisation calculation in
order to obtain a second NGWF set and kernel which correctly represent the
conduction manifold. Detail of the process may be found in the document
“Conduction NGWF optimisation and optical absorption spectra in ONETEP”[3]
and is published[4].

Two main parameters should be supplied: ``cond_energy_range`` which sets the
energy window above the HOMO in which conduction states will be optimised.
The second parameter cond energy gap
specifies the energy gap between the highest optimised state and the lowest un-
optimised state, this parameter is used to prevent attempts to optimise only
part of a set of degenerate states

Practical Example
=================

In this section we will discuss the procedure for running an EELS calculation
on a toy system: silene (the silicon equivalent of ethene). You will need to
obtain PAW pseudopotentials for Silicon and Hydrogen and the associated core
wavefunction data for Silicon. The author used the JTH pseudopotentials[5],
though can’t offer any guarantee of their suitability for any particular use.
A boilerplate input file for a onetep EELS calculation is provided in Appendix B.

This input file can also be downloaded from the tutorials section of the
onetep site. There are a few differences between this input and the one for a
single point calculation:

- PAW is mandatory
- We specify a second species for the atom whose core electrons we’re exciting
- Because a conduction calculation is being performed we must provide a species cond block
- We must provide a ``species_core_wf`` block and every species must be listed there.

The input file can be run swiftly on a single node and should produce a
large number of output files. Most of these files are ``.cube``’s of wavefunctions
produced by default during the properties calculations. The files of interest
are the ``.elnes_bin`` files, which contain OptaDoS compatible matrix elements.

A little more setup is needed before we can run OptaDoS (using the silene
example):

- A castep ``silene-out.cell`` file is needed, containing a symmetry block. ONETEP writes it with the EELS output (with k-point parallelism, under the base rootname, without the k-point group number); copy it to ``silene.cell`` if your version of OptaDoS reads that file
- By default two ``.elnes bin`` files are produced, one based on Kohn-Sham wavefunctions represented using only the valence NGWFs (``silene_val_grad.elnes_bin``) and a second which makes use of the joint basis of valence and conduction NGWFs (``silene_joint_grad.elnes_bin``)
- As per the discussion above, you should choose the latter and copy it to ``silene.elnes bin``.
- A ``silene.bands`` file must be produced, this is best done by copying ``silene.joint bands`` to ``silene.bands``.
- An OptaDoS input file, ``silene.odi`` is needed.

To assist in these tasks a utility script, ``prep_optados_eels``, is provided in the
utils-devel folder of the onetep distribution. Run it with the calculation seed name
as its argument and the steps listed above will be completed automatically,
for the joint basis:

::

    prep_optados_eels silene [joint|val|cond] [soc]

The optional second argument chooses the basis: ``joint`` (the default),
``val``, or ``cond`` for the unprojected conduction states (see below),
whose ``seed_cond_grad.elnes_bin`` and ``seed.proj_bands`` files are then
used. Add ``soc`` if the calculation used
``eigenstates_perturbative_soc``. For the ``cond`` basis and with ``soc``,
the ``.odi`` file has ``EFERMI : file``, otherwise ``EFERMI : insulator``.
The ``-out.cell`` file written by ONETEP is kept; if there is none, one is
made from the input file with an identity symmetry block.

The .odi file produced should be regarded as a basic template, consult the
OptaDoS documentation[6] if you wish to use more advanced features. Note
that at the moment only fixed broadening is supported by onetep.
When you are satisfied with your OptaDoS input file, execute OptaDoS
with your calculation seed name as the argument. All being well, you should
see a .dat file which you can plot with your favorite tool. Individual edges are
listed sequentially in the file, so a little post processing with awk or python is
needed to separate the edges for individual plotting

Unprojected conduction states
-----------------------------

EELS matrix elements are also calculated for the conduction NGWFs of the
unprojected route (``cond_unprojected : T``, or any conduction
calculation with ``edft : T``; see :ref:`cond_unprojected`), which
describe the valence and conduction states together. The matrix elements
are then written to ``seed_cond_grad.elnes_bin``, and the eigenvalues to
``seed.proj_bands``, which gives the number of valence electrons and the
valence Fermi level (rather than the occupancy of the window of conduction
states), so that OptaDoS can be used with ``EFERMI : file``.

Spin-orbit coupling
-------------------

With ``eigenstates_perturbative_soc : T`` (see
:ref:`eigenstates_perturbative_soc`), the matrix elements are those of
the :math:`2N` spinor eigenstates, for the valence, joint and unprojected
conduction bases. The core orbitals and the dipole operator do not depend
on spin, so for core orbital :math:`c` with core spin :math:`\sigma`

.. math::

   \langle c\,\sigma|\hat{\mathbf{O}}|\psi_n\rangle
   = \sum_\alpha \langle c|\hat{\mathbf{O}}|\phi_\alpha\rangle\,
   M^\sigma_{\alpha n} ,

where :math:`M^\uparrow` and :math:`M^\downarrow` are the coefficients of
the up and down copies of the NGWFs in the spinor eigenvector. In the
``.elnes_bin`` file each core orbital then appears twice, once for each
core spin, for a single spin component, with :math:`2N` bands, and the
``.bands`` file lists the spinor eigenvalues, each state holding one
electron. Since OptaDoS averages each edge over its core orbitals and
counts two electrons per state of a single spin component, this gives the
spectrum of the spinor states, which reduces to that of the
spin-unpolarised calculation in the limit of zero coupling.

When using OptaDoS with spinor states:

-  set ``EFERMI : file``: with the default (``optados``) or ``insulator``,
   OptaDoS assumes two electrons per state and places the Fermi level too
   low;
-  OptaDoS allows at most 7 core orbitals per edge, so the K (2 entries)
   and L\ :sub:`2,3` (6 entries) edges are available, but not
   M\ :sub:`4,5` (10 entries);
-  the core states are those of the scalar-relativistic PAW dataset,
   without spin-orbit splitting of the core level: L\ :sub:`2` and
   L\ :sub:`3` are not separated, and only the effect of SOC on the
   final states is included. For K edges this is complete.

For an insulator, the Fermi level in the ``.bands`` file is the highest
occupied eigenvalue, and OptaDoS includes states at the Fermi level in
an absorption spectrum, so a small artificial peak can appear at the
valence band maximum.

OptaDoS
=======

The OptaDoS code provides a single tool to compute densities of states and
optical spectra of various sorts. OptaDoS provides a number of smearing
schemes for evaluating integrals over the Brillouin Zone including fixed and
adaptive schemes. At present onetep only supports simple fixed broadening
schemes. The prep optados eels utility script provides a template .odi input
file:

::

    TASK : CORE
   
    # Method OptaDoS should use to work out
    # the Fermi energy in the material .
    # the insulator method relies on electron
    # counting and is best for the systems
    # onetep is most commonly used to study .
    EFERMI : insulator

    # Smearing scheme and width
    BROADENING : fixed
    DOS_SPACING : 0.01

    # Use an average of x , y and z components of
    # the position vector matrix elements
    CORE_GEOM : polycrystalline

    # Parameters below control lifetime and
    # instrument broadening
    CORE_LAI_BROADENING : true
    LAI_GAUSSIAN_WIDTH : 0.6
    LAI_LORENTZIAN_WIDTH : 0.2
    LAI_LORENTZIAN_SCALE : 0.1

References
==========

| [1] RJ Nicholls, AJ Morris, CJ Pickard, and JR Yates. Optados-a new tool
  for eels calculations. In Journal of Physics: Conference Series, volume 371,
  page 012062. IOP Publishing, 2012.
| [2] Peter E Blochl. Projector augmented-wave method. Physical Review B,
  50(24):17953, 1994.
| [3] Laura E. Ratcliff. Conduction NGWF optimisation and optical absorption
  spectra in onetep. http://www2.tcm.phy.cam.ac.uk/onetep/pmwiki/
  uploads/Main/Documentation/conduction.pdf, 2011. [Online; accessed
  22-Sept-2015].
| [4] Laura E Ratcliff, Nicholas DM Hine, and Peter D Haynes. Calculating op-
  tical absorption spectra for large systems using linear-scaling density func-
  tional theory. Physical Review B, 84(16):165131, 2011.
| [5] Francois Jollet, Marc Torrent, and Natalie Holzwarth. Generation of projec-
  tor augmented-wave atomic data: A 71 element validated table in the XML
  format. Computer Physics Communications, 185(4):1246 – 1254, 2014.
| [6] Andrew J. Morris, Rebecca J. Nicholls, Chris J. Pickard, and Jonathan R.
  Yates. OptaDoS user guide. http://www.cmmp.ucl.ac.uk/~ajm/
  optados/files/user_guide_1.0.pdf, 2014. [Online; accessed 22-Sept-
  2015]
