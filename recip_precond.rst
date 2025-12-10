========================================
Kinetic energy preconditioning in ONETEP
========================================

:Author: Chengcheng Xiao
:Date:   Dec 2025

ONETEP at the moment, by default use a modified Teter reciprocal
preconditioning.

`The original Teter reciprocal preconditioning function <https://doi.org/10.1103/PhysRevB.40.12255>`__
is defined as:

.. math::
   K_{\mathbf{G}, \mathbf{G}^{\prime}}=\delta_{\mathbf{G}, \mathbf{G}^{\prime}} \frac{27+18 x+12 x^2+8 x^3}{27+18 x+12 x^2+8 x^3+16 x^4}

where,

.. math::
   x=\frac{\left(\hbar^2|\mathbf{k}+\mathbf{G}|^2\right) / 2 m}{T_i^m}

and :math:`T_i^m=<\psi_i^m\left|\left(-\hbar^2 / 2 m\right) \nabla^2\right| \psi_i^m >` 
which is related to the band :math:`i` that :math:`K` is applied to.

In ONETEP, since we do not have the kinetic energy eigenvalues, :math:`x` is
modified to be:

.. math::
   x=\frac{\frac{1}{2}|\mathbf{G}|^2}{\frac{1}{2}k_0^2}

and :math:`k_0` is a manually specified number.

Automatic selection of k_0
==========================

Since ONETEP version 8, it is possible to perform calculations where :math:`k_0`
is calculated automatically based on the average kinetic energy at each NGWF
step. I.e.,

.. math::
   k_0 = \sqrt{\frac{\mathrm{Max[Tr(KT)]}}{n_\mathrm{occupied\ bands}}}

where :math:`\mathrm{Max[Tr(KT)]}` is the max value of the trace of the kinetic
energy operator matrix in the NGWF basis, and :math:`n_\mathrm{occupied\ bands}`
is the number of occupied bands.

It is also possible to perfrom NGWF-specific preconditioning so that no
band averaging is needed. In this case, each NGWF will have its own :math:`k_0`
value based on its own kinetic energy expectation value.
There are a few choices in terms of the kintetic energy to use for the
upreconditioning for each NGWF.

   1. The diagonal elements of :math:`KT`.
   2. The diagonal elements of :math:`S^{-1}T`.
   3. The diagonal elements of :math:`T`.

Keywords
========

-  ``precond_recip`` [Basic, bool, default ``F``\ ] Turn on reciprocal space
   kinetic energy preconditioning? [Incompatible with ``precond_real``]

-  ``precond_real`` [Basic, bool, default ``T``\ ] Turn on real space
   kinetic energy preconditioning? [Incompatible with ``precond_recip`` and 
   automatic :math:`k0` mode.]

-  ``precond_scheme`` [Basic string, default ``TETER``\ ] The type of kinetic
   energy preconditioning scheme to use. Options are: `BG`, `MAURI`, `TETER` or
   `NONE`.

-  ``k_zero`` [Basic, float 1/Bohr, default ``3.0``\ ] The :math:`k_0` value
   used in the kinetic energy preconditioning. If set to a negative value,
   ONETEP will automatically calculate the k0 value based on the average
   kinetic energy of the system.

-  ``precond_array`` [Basic, bool, default ``F``\ ] Turn on NGWF-specific
   preconditioning where each NGWF has its own k0 value based on its own kinetic
   energy expectation value? (requires ``k_zero`` to be negative).

-  ``precond_array_type`` [Basic string, default ``KT``\ ] The type of kinetic
   energy to use for NGWF-specific preconditioning. Options are: `KT`, `INVST`,
   and `T`. (requires ``precond_array`` to be ``T``).
