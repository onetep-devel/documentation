==========================
Performance considerations
==========================

:Author: Jacek Dziedzic, University of Southampton

How quickly ONETEP runs your calculations depends on a number of factors. 
Choosing the number of nodes, processes and threads has already been described in :ref:`running_parallel`.

Here we mention techniques that can be used in addition to an efficient parallel decomposition to improve performance.

.. _user_fast_sparse_to_dense:

Fast sparse-to-dense and dense-to-sparse conversions
====================================================

In many scenarios, ensemble DFT calculations being a notable example, ONETEP needs to convert between two matrix
representations -- a distributed sparse one ("SPAM3"), and a distributed dense one ("BLACS"). These conversions
happen multiple (hundreds) times in a run, and involve substantial data communication. There are two ways in
which ONETEP can do these -- conveniently termed "slow" and "fast".

The slow approach is the only approach available until ONETEP 6.1.21. Starting from ONETEP 6.1.22, a fast approach
is also available, *and is the default*. This means that no action is required on your part to use the fast approach.
The fast approach is up to several times faster (for this part of the calculation).

You can check which approach you are using by examining the timings printed out at the end of your calculation
(provided you are not using ``timings_level 0``). If your timings include ``sparse_spam3toblacs_real_fast`` and
``sparse_blacstospam3_real_fast`` -- you are using the fast approach. If instead you have ``sparse_spam3toblacs_real_slow``
and ``sparse_blacstospam3_real_slow`` -- you are using the slow approach.
 
To switch between the approaches use:
  - ``fast_dense_to_sparse T`` and ``fast_sparse_to_dense T`` -- for the fast approach,
  - ``fast_dense_to_sparse F`` and ``fast_sparse_to_dense F`` -- for the slow approach.

The fast approach is currently incompatible with the image comms subsystem needed by NEB. This means you cannot
use the fast approach with NEB. Please add ``fast_dense_to_sparse F`` and ``fast_sparse_to_dense F`` to your input
file when using NEB.


.. _user_fast_density:

Fast density calculation (for users)
====================================

This is a user-level explanation -- for developer-oriented material, see :ref:`dev_fast_density`.

Calculating the electronic charge density is one of the more time-consuming operations in ONETEP. In a typical
calculation it has to be performed hundreds of times. There are two ways in which ONETEP can calculate the density
-- conveniently termed "slow" and "fast".

The slow approach is the only approach available until ONETEP 7.1.7. Starting from ONETEP 7.1.8, a fast approach
is also available, *but it is not the default*. This means that action is required on your part to use the fast approach.
The fast approach is up to several times faster (for this part of the calculation).

The fast approach works best for "serious" systems, it's not meant to address scenarios with KE cutoffs
below 700-800 eV or NGWFs smaller than 8.0 a0.

To switch between the approaches use:
  - ``fast_density T`` -- for the fast approach,
  - ``fast_density F`` -- for the slow approach.

Be aware that the fast approach *is an approximation*. The approximation is well-controllable, meaning you can get
as close with the accuracy to the slow (exact) approach as you like, albeit sacrificing performance as you do so.
Conversely, you can make the fast approach as fast as you like, but you will be sacrificing accuracy as you do so,
up to a point of making your results worthless. This means care must be taken when changing the parameters of
this approach -- non-expert users are advised to use the defaults, or even "safe settings" described below.

You can check which approach you are using by examining the timings printed out at the end of your calculation
(provided you are not using ``timings_level 0``). If your timings include ``density_on_dbl_grid_fast`` and
``density_fast_new_ngwfs`` -- you are using the fast approach. If instead you have ``density_on_dbl_grid``
and ``density_batch_interp_deposit`` -- you are using the slow approach.

The main idea behind the fast density approach is *trimming* interpolated NGWFs, that is, ignoring the points
where their values are below a prescribed threshold. The value of this threshold, set by ``fast_density_trim_threshold``
is the main parameter controlling the balance between accuracy and efficiency. 
It is also independent of NGWF radii. The default is ``2E-6``.
The parameter already includes grid weights, so you *do not* need to adjust it when changing ``psinc_spacing`` or
``cutoff_energy``. To make your calculation more accurate, decrease the threshold -- probably not below ``1E-7``.
To make your calculation faster, increase the threshold -- probably not above ``1E-5``. 

If you use ``fast_density_output_detail VERBOSE`` (or higher), ONETEP will print out an estimate of the accuracy
of the approximation every time NGWFs change. It will look like something like :numref:`Figure fig:fast_density_info`,
see the *accuracy of approximation* line. This tells you to how many digits the approximated NGWF charge is equal
to the exact (double FFT-box) NGWF charge, in the root-mean-square sense over all NGWFs in the system. In this
example our approximated charge is no further from 1.0 than by 1E-7 (and is slightly closer, because we got 7.3,
not 7.0).

As your calculation progresses, this value will fluctuate, and is likely go down slightly, as the NGWFs become
more diffuse. As a rule of thumb, if it gets below 5.0-6.0, you will have difficulty converging NGWFs to the
default threshold. If it is above 9.0, you are probably using too much accuracy, losing efficiency as you do that.

.. _Figure fig:fast_density_info:
.. figure:: _static/resources/fast_density_fig_1.png
   :alt: Fast density -- information on accuracy and memory use.
   :name: fig:fast_density_info
   :width: 75.0%
   :target: _static/resources/fast_density_fig_1.png

   The summary printed by fast density every time the NGWFs change. Of main interest are: *accuracy of approximation* (shown
   in red) and *estimated high-memory watermark per MPI rank* (shown in yellow).

Another notable quantity in :numref:`Figure fig:fast_density_info` is the *estimated high-memory watermark per MPI rank*
(shown in yellow). This is a reminder that the fast density approach uses significantly more memory than the slow approach.
The value in the printout is the expected *maximum* memory that fast density uses *per MPI rank*. If your printout is
truncated before you reached this line, you most likely already ran out of memory. At this stage, we use an all-or-nothing
approach -- there is no way to give the algorithm a memory allowance and tell it that it should not consume more. Work on
this is in progress. The best way to reduce memory load is to use fewer processes per node and more threads. If this is
not sufficient, you can reduce the memory load by using more nodes, but this is not a linear dependence -- i.e. you will
*not* reduce the load by a factor of two if you add twice as many nodes. Finally, note that what is printed out is the
amount of memory consumed by the fast density approach, not by all of ONETEP.

When is fast density used?
--------------------------

Fast density is only used for energy evaluations done from ``hamiltonian_mod`` -- via ``hamiltonian_lhxc_calculate()``
and ``hamiltonian_energy_components()``. These are the costly density calculations, because they are done hundreds
of times in the course of a calculation. All other density calculations (done in forces, properties, eigenstates, 
linear response, lr_tddft, population, dma, dmft, EDA, implicit solvent restarts) are always done using the exact
(slow) method. The rationale is that these are done much less often and possibly require more accuracy.

If you want to know when the fast and slow routines are called, specify ``fast_density_output_detail PROLIX``
or higher.

More accuracy
-------------

The default settings should give you sufficient accuracy to converge NGWFs to the default threshold and to get energies and 
forces that are negligibly different from those obtained with the slow approach. However, for more difficult systems,
particularly if using low kinetic energy cutoffs (say, below 700 eV -- like would probably be used with PAW), 
you might need to adjust the parameters to get desired accuracy.

In addition to adjusting ``fast_density_trim_threshold`` down (to perhaps 1E-6 or 5E-7), you may want to use 
``fast_density_off_for_last T`` (the default is ``F``). This will tell ONETEP to use the slow (but exact) approach for
the final energy evaluation. You will know this happened by examining the output file and looking for:

::

  ! Looks like the last energy evaluation.
  ! The fast density calculation will now be disabled in the interest of accuracy.

Note that this will not be printed if ``fast_density_output_detail`` is ``BRIEF`` or if fast density would already
have been switched off by ``fast_density_elec_energy_tol`` (see below). This setting resets any time you start a new
NGWF convergence loop -- that means that in auto solvation, geometry optimisation, MD, etc. each optimisation will
start with fast density turned on.

Also note that this switching is done in the NGWF convergence loop. If you are working with fixed NGWFs
(``maxit_ngwf_cg 0`` (or negative)), this switching will not take place.

Furthermore, particularly if your calculation struggles to converge to the default
NGWF threshold, you can set ``fast_density_elec_energy_tol``. This is the energy change per atom between NGWF steps
below which ONETEP will switch to the slow (but exact) approach. It's the same quantity that is used as the energy
convergence criterion in ``elec_energy_tol``. The default is ``1E-50``, effectively turning this off. Setting it
to ``1E-7`` will typically have ONETEP switch to the slow approach for the last few NGWF iterations. The higher
you set this, the sooner ONETEP will switch to the slow approach. This, of course, eats into your efficiency gain.
You will know if and when this happened by examining the output file and looking for:

::

  ! Energy change per atom: 0.30287E-07 Eh < 0.10000E-06.
  ! The fast density calculation will now be disabled in the interest of accuracy.

Note that this will not be printed if ``fast_density_output_detail`` is ``BRIEF``. This setting resets any time 
you start a new NGWF convergence loop -- that means that in auto solvation, geometry optimisation, MD, etc. each 
optimisation will start with fast density turned on.

Note that you need at least two NGWF iterations to have a meaningful energy change to examine, so this setting
has no effect if you take fewer than two NGWF iterations.

Remaining options
-----------------

The default output detail of fast density is the same as specified for ``output_detail``. You can set it separately
by specifying ``fast_density_output_detail``. The available options are the same as for all ONETEP output details:
``BRIEF``, ``NORMAL``, ``VERBOSE``, ``PROLIX`` and ``MAXIMUM``.

If, in the future, other methods of trimming NGWFs than by using a fixed threshold become available, you will be
able to use ``fast_density_trim_by`` to control these. Currently the only supported option is ``VALUE``.

Example settings
----------------

For a quick-and-dirty calculation use: 
 - ``fast_density T``
 - ``fast_density_threshold 2E-5``.

For a typical calculation just use: 
 - ``fast_density T`` (which will use the default of ``fast_density_threshold 2E-6``).

For an accurate, but slower calculation use:
 -  ``fast_density T``
 - ``fast_density_threshold 1E-6``
 - ``fast_density_off_for_last T``
 - ``fast_density_elec_energy_tol 1E-7``.

For very safe settings that should provide a modest gain in efficiency, try:
 - ``fast_density T``
 - ``fast_density_threshold 5E-7``
 - ``fast_density_off_for_last T``
 - ``fast_density_elec_energy_tol 3E-7``.
 
Compatilibity
-------------

Fast density is known to work (to the best of our knowledge) with the following additional functionalities:
  - extended NGWFs,
  - PBCs and OBCs,
  - implicit solvation,
  - hybrid functionals and Hartree-Fock exchange,
  - ``fine_grid_scale`` larger than ``2.0``,
  - PAW,
  - DFT+U,
  - conduction,
  - MD,
  - geometry optimisation,
  - TS search,
  - NEB,
  - EDFT and LNV.
  

Fast density is known *not* to work (this we know with certainty) with the following additional functionalities:
  - complex NGWFs,
  - TD-DFT (mixed bases are not supported at this point).
  - EMFT (regions).

ONETEP will stop with an error if either of these is used with `fast_density T`.
