=============================================
Empirical Dispersion Correction
=============================================

:Author: Max Phipps, University of Southampton
:Author: Arihant Bhandari, University of Southampton
:Author: Gianluca Seaford, University of Warwick

:Date:   October 2026

ONETEP offers two families of empirical dispersion correction: four
built-in pairwise models (Elstner, two variants of Wu-Yang, and
Grimme's D2); and Grimme's D3 and D4 dispersion corrections implemented
via the simple-DFTD3 and DFTD4 external libraries respectively
and distributed with ONETEP.

Theory
======

Energy
------

Both families of dispersion corrections add an empirical correction
to the Kohn-Sham total energy,

.. math::
   :label: disp_corr

   E_{DFT-D} = E_{KS-DFT} + E_{disp}

The leading dispersion energy correction to :math:`E_{disp}`, common to
all models, is a damped pairwise sum of :math:`C_6/R^6` terms over atoms
and their periodic images [Vasp]_, [Hill2009]_:

.. math::
   :label: E^{(6)}_{disp}

   E^{(6)}_{disp} = -\frac{1}{2}\cdot s_6\cdot \sum^{N_{at}}_{i=1} \sum^{N_{at}}_{j=1}
   \sum_{L}^* \dfrac{C_{6,ij}}{R^6_{ij}} f_{damp}( R_{ij} ) g_{smooth}(R_{ij})

Here:

-  :math:`s_6` is a density functional dependent global scaling factor,

-  :math:`C_{6,ij}` is a dispersion coefficient for the atom pair
   :math:`ij`,

-  :math:`L` denotes all translations of the unit cell within the van
   der Waals radial cutoff :math:`R_{cut}`,

-  :math:`*` denotes :math:`i\ne j` in :math:`L=0`

-  :math:`R_{ij}` is the distance :math:`| \overrightarrow{R^{0}_{i}} -
   \overrightarrow{R^L_{j}} |` between atom i in the parent cell
   :math:`L=0` and the atom j in all possible translations :math:`L`,

-  :math:`f_{damp}(R_{ij})` is a damping function that is unity at large
   distances and zero at small
   distances [Hill2009]_, [Grimme2006]_; for the order-:math:`n` terms of
   Grimme's D3 and D4 corrections it is written :math:`f_{damp,n}(R_{ij})`,

-  :math:`g_{smooth}(R_{ij})` is a smoothing function for truncating the
   interactions beyond van der Waals radial cutoff.

For Grimme's D3 corrections, the :math:`C_{6,ij}` coefficients depend on
each atom's coordination number [Grimme2010]_, while Grimme's D4
corrections have an additional dependence on atomic partial charges from
an electronegativity-equilibration model [Caldeweyher2019]_.

Grimme's D3 and D4 corrections also include both an additional pairwise
:math:`C_{8,ij}` term and an optional three-body Axilrod-Teller-Muto (ATM)
term :eq:`E^{ATM}_{disp}`, and thus

.. math::
   :label: E_disp_d3d4

   E_{disp} = E^{(6)}_{disp} + E^{(8)}_{disp} + E^{ATM}_{disp}

where

.. math::
   :label: E^{(8)}_{disp}

   E^{(8)}_{disp} = -\frac{1}{2}\cdot s_8\cdot \sum^{N_{at}}_{i=1} \sum^{N_{at}}_{j=1}
   \sum_{L}^* \dfrac{C_{8,ij}}{R^8_{ij}} f_{damp,8}( R_{ij} ) g_{smooth}(R_{ij})

for some functional-dependent scaling factor :math:`s_8`. Here,
:math:`C_{8,ij} = 3\sqrt{Q_i Q_j}\,C_{6,ij}` is obtained from the
:math:`C_{6,ij}` coefficients and

.. math::
   :label: Q_i

   Q_i = \tfrac{1}{2}\sqrt{Z_i}\,\langle r^4\rangle_i/\langle r^2\rangle_i.

The Axilrod-Teller-Muto term is given by

.. math::
   :label: E^{ATM}_{disp}

   E^{ATM}_{disp} = s_9 \sum_{ijk} C_{9,ijk}\,A_{ijk}\,f_{ijk},
   \qquad
   A_{ijk} = \frac{3\cos\theta_{ij}\cos\theta_{jk}\cos\theta_{ik} + 1}
                  {\left(R_{ij}R_{jk}R_{ik}\right)^3},
   \qquad
   f_{ijk} = \frac{1}{1 + 6\left(\bar{R}_{0,ijk}/\bar{R}_{ijk}\right)^{16}}

where the sum runs over unique triples of atoms, including periodic
images within :math:`R_{cut}`. Here, :math:`\theta_{ij}` denotes the angle
formed between atoms :math:`i`, :math:`j`, and :math:`k` that lies
opposite the side :math:`R_{ij}` in the summation;
:math:`C_{9,ijk} = \sqrt{C_{6,ij}C_{6,ik}C_{6,jk}}`; and
:math:`\bar{R}_{ijk}` is the geometric mean of the three distances.
Additionally, :math:`\bar{R}_{0,ijk}` denotes the geometric mean of the
pair radii: for Grimme's D3 corrections a value of :math:`\tfrac{4}{3}`
times the tabulated cutoff radii :math:`R_{0,ij}` is used, while Grimme's
D4 corrections use the Becke-Johnson radii
:math:`a_1\sqrt{C_{8,ij}/C_{6,ij}} + a_2`. The :math:`s_9` parameter is 1
when the three-body term is included and 0 otherwise, and the three-body
term uses a cutoff that is the smaller of :math:`R_{cut}` and 40 bohr.

For the in-built dispersion corrections, :math:`E^{(8)}_{disp}` and
:math:`E^{ATM}_{disp}` are zero, and thus

.. math::
   :label: E_disp_inbuilt

   E_{disp} = E^{(6)}_{disp}

where the dispersion coefficients and the form of the damping function
are dependent upon the desired empirical vdW correction model. Damping
functions for the in-built dispersion corrections are given below:

.. list-table::
   :widths: 14 30 32 20
   :header-rows: 1

   * - Dispersion Correction
     - Description
     - Damping function :math:`f_{damp}( R_{ij} )`
     - vdW radii (:math:`R_{0,ij}`)
   * - ``ELSTNER``
     - Elstner's damping function [Elstner2001]_.
     - :math:`(1-\exp(-c_{damp}(R_{ij}/R_{0,ij})^7))^4`
     - :math:`\frac{{R_{0,i}}^3+{R_{0,j}}^3}{{R_{0,i}}^2+{R_{0,j}}^2}`
   * - ``WUYANG1``
     - First Wu-Yang damping function [Wu2002]_.
     - :math:`(1-\exp(-c_{damp}(R_{ij}/R_{0,ij})^3))^2`
     - :math:`\frac{{R_{0,i}}^3+{R_{0,j}}^3}{{R_{0,i}}^2+{R_{0,j}}^2}`
   * - ``WUYANG2``
     - Second Wu-Yang damping function [Wu2002]_.
     - :math:`\frac{1}{1+\exp(-c_{damp}(R_{ij}/R_{0,ij}-1))}`
     - :math:`\frac{{R_{0,i}}^3+{R_{0,j}}^3}{{R_{0,i}}^2+{R_{0,j}}^2}`
   * - ``GRIMMED2``
     - Grimme's D2 dispersion correction [Grimme2006]_.
     - :math:`\frac{1}{1+\exp(-c_{damp}(R_{ij}/R_{0,ij}-1))}`
     - :math:`{R_{0,i}}+{R_{0,j}}`

for some damping constant :math:`c_{damp}` (referred to as :math:`d`
within the literature of Grimme [Grimme2006]_ and
:math:`c_{damp}` by Hill [Hill2009]_) and
:math:`R_{0,ij}` is determined by the vdW radii of the atomic pair
:math:`i` and :math:`j`.

The smoothing function for in-built dispersion corrections has the
following form:

.. math:: g_{smooth}(R_{ij})=1-e^{-\left(R_{ij}-R_{cut}\right)^2}

where :math:`R_{cut}` is the radial cutoff for van der Waals interactions,
and can be set by the following keyword in the input file:

``vdw_radial_cutoff: x units`` [Real physical, default
``vdw_radial_cutoff: 100 bohr``].

For Grimme's D3 and D4 corrections, both the scaling factors :math:`s_6`,
:math:`s_8` and the damping parameters are dependent on the
exchange-correlation functional, and are taken from the simple-DFTD3
[Ehlert2024]_ and DFTD4 libraries respectively. The damping functions are
applied to each pairwise term, with :math:`\alpha_6 = 14` and
:math:`\alpha_8 = \alpha_6 + 2`, and are listed below:

.. list-table::
   :widths: 14 30 32 20
   :header-rows: 1

   * - Dispersion Correction
     - Description
     - Damping function :math:`f_{damp,n}( R_{ij} )`
     - vdW radii (:math:`R_{0,ij}`)
   * - ``D3``
     - Grimme's zero damping function [Grimme2010]_.
     - :math:`\frac{1}{1+6\left(s_{r,n}R_{0,ij}/R_{ij}\right)^{\alpha_n}}`
     - Tabulated pairwise cutoff radii [Grimme2010]_
   * - ``D3BJ``
     - Becke-Johnson damping function [Grimme2011]_.
     - :math:`\frac{R_{ij}^n}{R_{ij}^n+\left(a_1R_{0,ij}+a_2\right)^n}`
     - :math:`\sqrt{C_{8,ij}/C_{6,ij}}`
   * - ``D3M``
     - Modified zero damping function [Smith2016]_.
     - :math:`\frac{1}{1+6\left(R_{ij}/(s_{r,n}R_{0,ij})+\beta R_{0,ij}\right)^{-\alpha_n}}`
     - Tabulated pairwise cutoff radii [Grimme2010]_
   * - ``D3MBJ``
     - Modified Becke-Johnson damping function [Smith2016]_.
     - :math:`\frac{R_{ij}^n}{R_{ij}^n+\left(a_1R_{0,ij}+a_2\right)^n}`
     - :math:`\sqrt{C_{8,ij}/C_{6,ij}}`
   * - ``D4``, ``D4S``
     - Becke-Johnson damping function [Caldeweyher2019]_.
     - :math:`\frac{R_{ij}^n}{R_{ij}^n+\left(a_1R_{0,ij}+a_2\right)^n}`
     - :math:`\sqrt{C_{8,ij}/C_{6,ij}}`

where :math:`s_{r,n}`, :math:`\beta`, :math:`a_1` and :math:`a_2` are
functional-dependent damping parameters.

Forces
------

During geometry optimization, force corrections are calculated from the
derivative of the dispersion energy with respect to the ionic coordinates
:math:`\{\mathbf{s}_i\}`. For a given dispersion correction, the
corresponding force correction can be expressed as:

.. math::
   :label: F_total

   \mathbf{F}_i = -\frac{dE_{disp}}{d\mathbf{s}_i}

For the four in-built dispersion corrections, the total force correction
is thus

.. math::
   :label: F_total_inbuilt

   \mathbf{F}_i = -\frac{\partial E^{(6)}_{disp}}{\partial \mathbf{s}_i} = s_6
   \sum^{N_{at}}_{j=1} \sum_{L}^* \dfrac{C_{6,ij}}{R^6_{ij}}
   \left[f_{damp}(R_{ij})\,g'_{smooth}(R_{ij}) + g_{smooth}(R_{ij})
   \left(f'_{damp}(R_{ij}) - \frac{6f_{damp}(R_{ij})}{R_{ij}}\right)\right]
   \frac{\partial R_{ij}}{\partial \mathbf{s}_i}

However, for Grimme's D3 and D4 corrections, the resulting correction
to the forces is more complicated, with additional contributions from
:math:`E^{(8)}_{disp}`, :math:`E^{ATM}_{disp}` (if present), and the
position dependence of the dispersion coefficients:

.. math::
   :label: F_total_d3d4

   \mathbf{F}_i = -\left( \frac{dE^{(6)}_{disp}}{d\mathbf{s}_i}
                  + \frac{dE^{(8)}_{disp}}{d\mathbf{s}_i}
                  + \frac{dE^{ATM}_{disp}}{d\mathbf{s}_i} \right)

Here, the :math:`C_{6,ij}` term is given by :eq:`F_total_inbuilt` when
evaluated with :math:`g_{smooth} = 1`, ensuring :math:`g'_{smooth} = 0`,
and :math:`f_{damp} = f_{damp,6}`. Hence, the force correction contains
three additional contributions.

1. A contribution originating with the additional :math:`C_{8,ij}` term
   in :eq:`E^{(8)}_{disp}`, with similar form to the :math:`C_{6,ij}`
   term:

   .. math::
      :label: F^{(8)}_{disp}

      \frac{\partial E^{(8)}_{disp}}{\partial \mathbf{s}_i}\bigg|_{CN,q} =
      -s_8 \sum^{N_{at}}_{j=1} \sum_{L}^* \dfrac{C_{8,ij}}{R^8_{ij}}
      \left(f'_{damp,8}(R_{ij})
      - \frac{8f_{damp,8}(R_{ij})}{R_{ij}}\right)
      \frac{\partial R_{ij}}{\partial \mathbf{s}_i}

   Here :math:`\vert_{CN,q}` denotes a derivative taken at fixed
   coordination numbers and charges, i.e. at fixed dispersion
   coefficients.

2. A contribution due to the dependence of :math:`C_{6,ij}` coefficients
   on the coordination numbers :math:`CN_k` and, for Grimme's D4
   corrections, on the atomic partial charges :math:`q_k`, which both
   change as the atoms move. Applying the chain rule yields:

   .. math::
      :label: F^{CN,q}_{disp}

      \frac{dE_{disp}}{d\mathbf{s}_i} =
      \frac{\partial E_{disp}}{\partial \mathbf{s}_i}\bigg|_{CN,q}
      + \sum^{N_{at}}_{k=1} \left(
      \frac{\partial E_{disp}}{\partial CN_k}
      \frac{\partial CN_k}{\partial \mathbf{s}_i}
      + \frac{\partial E_{disp}}{\partial q_k}
      \frac{\partial q_k}{\partial \mathbf{s}_i}
      \right)

   where the first term is the sum of the explicit contributions, and the
   third term is present for Grimme's D4 corrections only. For Grimme's
   D3 corrections, the coordination number

   .. math::

      CN_k = \sum_{j \ne k} \frac{1}{1 + e^{-k_1\left(k_2
      (R_{cov,k}+R_{cov,j})/R_{kj} - 1\right)}}

   is a summation over neighbouring atoms and their periodic images, with
   covalent radii :math:`R_{cov}`, :math:`k_1 = 16` and
   :math:`k_2 = 4/3` [Grimme2010]_. Grimme's D4 corrections use an
   error-function counting function weighted by electronegativity
   differences, with the charges :math:`q_k` obtained from an
   electronegativity-equilibration model [Caldeweyher2019]_.

3. A contribution from the three-body ATM term, if included, which arises
   from the explicit dependence of :eq:`E^{ATM}_{disp}` on the interatomic
   distances, of which only :math:`R_{ij}` and :math:`R_{ik}` depend on
   the position of atom :math:`i`:

   .. math::
      :label: F^{ATM}_{disp}

      \frac{\partial E^{ATM}_{disp}}{\partial \mathbf{s}_i}\bigg|_{CN,q}
      = s_9 \sum_{jk} C_{9,ijk} \sum_{ab \in \{ij,\,ik\}}
      \left(\frac{\partial A_{ijk}}{\partial R_{ab}}\, f_{ijk}
      + A_{ijk}\,\frac{\partial f_{ijk}}{\partial R_{ab}}\right)
      \frac{\partial R_{ab}}{\partial \mathbf{s}_i}

   where :math:`A_{ijk}` and :math:`f_{ijk}` are the angular and damping
   factors of :eq:`E^{ATM}_{disp}`, with

   .. math::

      \frac{\partial f_{ijk}}{\partial R_{ab}} =
      \frac{16}{3}\,\frac{f_{ijk}\,(1 - f_{ijk})}{R_{ab}}

   The dependence of :math:`C_{9,ijk}` on the coordination numbers and
   charges is included in contribution 2.

All of these contributions are computed analytically by the simple-DFTD3
and DFTD4 libraries.

Activating the dispersion corrections
=====================================

Activation of dispersion corrections is achieved using the

``dispersion``

keyword followed by the relevant dispersion model name. For example,
Grimme's D2 correction [Grimme2006]_ would be selected using:

``dispersion : GRIMMED2``

The exchange-correlation functionals available with optimized parameters
for the dispersion models are given below:

.. list-table::
   :widths: 20 70
   :header-rows: 1

   * - Keyword
     - Available XC functionals
   * - ``ELSTNER``
     - BLYP, PBE, PW91, REVPBE, RPBE, XLYP
   * - ``WUYANG1``
     - BLYP, PBE, PW91, REVPBE, RPBE, XLYP
   * - ``WUYANG2``
     - BLYP, PBE, PW91, REVPBE, RPBE, XLYP
   * - ``GRIMMED2``
     - BLYP, PBE, B3LYP
   * - ``D3``, ``D3-ATM``
     - BLYP, PBE, REVPBE, RPBE, PBESOL, XLYP, B1LYP, B3LYP, B3PW91, PBE0,
       X3LYP, HF, PKZB
   * - ``D3BJ``, ``D3BJ-ATM``
     - BLYP, PBE, REVPBE, RPBE, PBESOL, PW91, XLYP, B1LYP, B3LYP, B3PW91,
       PBE0, X3LYP, HF, RSCAN
   * - ``D3M``, ``D3M-ATM``,
       ``D3MBJ``, ``D3MBJ-ATM``
     - BLYP, PBE, B3LYP, PBE0
   * - ``D4``, ``D4-NOATM``,
       ``D4S``, ``D4S-NOATM``
     - BLYP, PBE, REVPBE, RPBE, PBESOL, AM05, PW91, XLYP, B1LYP, B1PW91,
       B3LYP, B3PW91, PBE0, X3LYP, HF, RSCAN

The :math:`C_{6,ij}` coefficients and the coefficients of the Elstner and
Wu-Yang damping functions are optimized for each xc
functional to minimize the root mean square difference of the
interaction energy from the literature values for a selection of
dispersion-dominant biological complexes from the JSCH-2005 and S22
sets [Jurecka2006]_ and the database of
Morgado [Morgado2007]_. For the D2 correction of
Grimme the parameters are optimized as described in the
literature [Grimme2006]_.

The :math:`s_6` parameters are unity for the Elstner and Wu-Yang models
and are fitted by least squares optimization of interaction energy error
for 40 noncovalently bound complexes for the D2 correction of Grimme.

In the case of using an unoptimized functional not given within the
list, default unoptimized :math:`c_{damp}`, :math:`C_{6,ij}`, and
:math:`R_{0,i}` values are adopted for the damping functions of Elstner
or Wu and Yang as described by Hill [Hill2009]_ and a
default :math:`s_6` value of 1.00 is adopted for Grimme's D2 correction
model.

Grimme's D3 dispersion corrections are available in both zero damping
[Grimme2010]_ (``D3``) and Becke-Johnson damping [Grimme2011]_ (``D3BJ``)
forms, as well as the respective modified variants ``D3M`` and ``D3MBJ``
outlined in [Smith2016]_. The three-body term is not included by default
(:math:`s_9 = 0`); it is included by appending the suffix ``-ATM`` to the
correction name (e.g. ``D3-ATM``, :math:`s_9 = 1`).

Grimme's D4 dispersion corrections are available in both smoothed
(``D4S``) [Tkachenko2024]_ and non-smoothed (``D4``) variants. These
include the three-body term by default (:math:`s_9 = 1`), which can be
disabled by appending the ``-NOATM`` suffix (:math:`s_9 = 0`).

Hybrid functionals require the setup described in :doc:`hfx`, and
meta-GGAs that described in :doc:`mgga`. If no D3/D4 parameters exist for
the chosen functional and model (for example LDA, or ``GGA``, for which
``RPBE`` should be given explicitly), ONETEP stops with an error naming
the functional.

Both the D3 and D4 dispersion corrections can be used with ``LIBXC``
functionals. With ``xc_functional : LIBXC``, the functional is identified
from ``libxc_x_func_id`` and ``libxc_c_func_id``, although not all
combinations are currently supported by the simple-DFTD3 library.

Grimme's D3 and D4 corrections use the boundary conditions set by
``vdw_bc`` (see below), and Grimme's D4 corrections use the total system
``charge`` in their charge model. The D3/D4 contribution to the stress
tensor is not yet included.

.. note::
   DFT-D3/D4 cannot be combined with nonlocal van der Waals functionals
   (``VDWDF``, ``VDWDF2``, ``VDWC09``, ``OPTPBE``, ``OPTB88``, ``PBEK1``,
   ``VV10``, ``VV10BC``, ``AVV10S``, or libxc functionals containing
   VV10), since dispersion would then be counted twice; ONETEP stops with
   an error if this is requested.


Overriding dispersion correction parameters
===========================================
.. note::
   These overrides apply only to the in-built dispersion corrections; the
   D3/D4 parameters are taken from the simple-DFTD3 and DFTD4 libraries and
   cannot be changed.

It is possible to override the default parameters of the in-built dispersion
damping functions. This option allows the user to specify parameters for
elements and functionals for which values are not given. The
atom-dependent variables :math:`C_{6,i}` (used to calculate
:math:`C_{6,ij}`), :math:`R_{0,i}` (related to the atomic vdW radius of
an atom :math:`i`), and :math:`n_{eff}` (used in the calculation of
:math:`C_{6,ij}` for all damping functions excluding the D2 correction
of Grimme) are modified using the

``vdw_params``

block. This override block applies the parameter changes to atoms by
their atomic number (nzatom). eg. To override the dispersion parameters
associated with nitrogen,

::

    %block vdw_params
    ! nzatom, c6coeff, radzero, neff
      7       21.1200  2.6200   2.5100
    %endblock vdw_params

To override the damping constant :math:`c_{damp}` associated with a
damping function, the keyword

``vdw_dcoeff``

followed by the modified damping constant parameter is used. eg.

``vdw_dcoeff 11.0``

Boundary Conditions
===================

The boundary conditions are set by the following keyword:

``vdw_bc P/O P/O P/O``

which accepts a string which should contain three characters (which may
be separated by spaces), specifying the BCs along the :math:`x`,
:math:`y` and :math:`z` directions of the simulation cell. ‘P’ for
periodic and ‘O’ for open. In case, the keyword is not specified, BCs
set same as

``ion_ion_bc``. The same boundary conditions are used for Grimme's D3
and D4 corrections.

Compilation and linking
=======================

Grimme's D3 and D4 corrections are optional at compile time, and are
enabled by adding the following flags to ``FFLAGS`` in your
``config/conf.*`` file:

 - ``-DDFTD3`` to enable Grimme's D3 corrections (simple-DFTD3),
 - ``-DDFTD4`` to enable Grimme's D4 corrections (DFTD4).

The libraries, distributed in ``dftd/``, are then compiled as part of the
usual build, using the ``F90`` and ``OPTFLAGS`` of the config file, and
with OpenMP if ``FFLAGS`` contains an OpenMP flag. The following optional
settings can also be added to the config file:

 - ``DFTD_OPENMP = OFF`` to compile the libraries without OpenMP,
 - ``DFTD_FFLAGS`` to pass extra compiler flags to the libraries.

If a D3 or D4 correction is requested from a binary compiled without the
corresponding flag, ONETEP stops with an error while checking the input.

[Vasp] https://www.vasp.at/wiki/index.php/DFT-D2

[Hill2009] Q. Hill and C-K Skylaris, *Proc. R. Soc. A* 465(2103):669-683, 2009

[Grimme2006] S. Grimme, *J. Comput. Chem.* 27(15):1787-1799, 2006

[Elstner2001] M. Elstner, P. Hobza, T. Frauenheim, S. Suhai and E. Kaxiras, *J. Chem. Phys.* 114(12):5149-5155, 2001

[Wu2002] Q. Wu and W. Yang, *J. Chem. Phys.*, 116(2):515-524, 2002

[Jurecka2006] P. Jurečka, J. Šponer, J. Černý and P. Hobza, *Phys. Chem. Chem. Phys.* 8:1985-1993, 2006

[Morgado2007] C. A. Morgado, J. P. McNamara, I. H. Hillier, N. A. Burton and M. A. Vincent *J. Chem. Theory Comput.* 3(5):1656-1664, 2007

[Smith2016] D. G. A. Smith, L. A. Burns, K. Patkowski and C. D. Sherrill, *J. Phys. Chem. Lett.* **7**, 2197 (2016)

[Ehlert2024] S. Ehlert, *J. Open Source Softw.* **9**, 7169 (2024)

[Caldeweyher2017] E. Caldeweyher, C. Bannwarth and S. Grimme, *J. Chem. Phys.* **147**, 034112 (2017)

[Caldeweyher2019] E. Caldeweyher, S. Ehlert, A. Hansen, H. Neugebauer, S. Spicher, C. Bannwarth and S. Grimme, *J. Chem. Phys.* **150**, 154122 (2019)

[Caldeweyher2020] E. Caldeweyher, J.-M. Mewes, S. Ehlert and S. Grimme, *Phys. Chem. Chem. Phys.* **22**, 8499 (2020)

[Grimme2010] S. Grimme, J. Antony, S. Ehrlich and H. Krieg, *J. Chem. Phys.* **132**, 154104 (2010)

[Grimme2011] S. Grimme, S. Ehrlich and L. Goerigk, *J. Comput. Chem.* **32**, 1456 (2011)

[Tkachenko2024] N. V. Tkachenko, L. B. Dittmer, R. Tomann and M. Head-Gordon, *J. Phys. Chem. Lett.* **15**, 10629 (2024)
