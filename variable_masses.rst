=============================================
Variable Ionic Masses
=============================================

:Author: Harry Brough, University of Manchester

Introduction
===================================

ONETEP [Skylaris2005]_ assigns every ionic species a default mass, taken
from an internal table and determined by the element. The ``species_mass`` block, introduced in v8.2, overrides
these defaults on a per-species basis, allowing isotopic substitution without altering the electronic structure of the
calculation.

This is of use for:

* **Molecular dynamics** — isotope effects on diffusion and rates of
  barrier crossing.
* **Vibrational and phonon calculations** — isotopic shifts of harmonic
  frequencies, zero-point energies, thermochemical corrections, and
  hence kinetic isotope effects.

Because the Born-Oppenheimer potential energy surface does not depend on
the ionic masses, no aspect of the electronic structure calculation is
changed: total energies, forces, NGWFs, the density kernel and the
locations of stationary points are all identical to those obtained with
the default masses. Only mass-dependent nuclear properties are affected.

Keywords
===================================

The atomic masses can be specified with the ``species_mass`` block keyword, as shown:

::

    %block species_mass
    H    2.014102
    C    13.00335
    O    17.99916
    %endblock species_mass


This requests a calculation with deuterium instead of protium, and with :sup:`13`\ C and :sup:`18`\ O nuclei. The
first entry on each line is the species label, exactly as it appears in the first column of the ``species`` block.

Not every atomic species needs to be listed in ``species_mass``; any species omitted retains its default mass.

By default, the mass units are atomic mass units (amu) — this can be changed in the same way as other block keywords. Other possible units are grams (g) and kilograms (kg).

Selective Isotopic Substitution
=============================================

Because ``species_mass`` acts on species labels rather than on elements,
distinct labels must be introduced in the ``species`` block in order to
substitute only some atoms of a given element. To deuterate a single
hydroxyl hydrogen, for instance:

::

    %block species
    H    H   1   1   8.0
    H_D  H   1   1   8.0
    O    O   8   4   8.0
    %endblock species

    %block species_mass
    H_D  2.014102
    %endblock species_mass

Both hydrogen labels share the same element, atomic number and
pseudopotential, and only the mass differs. Atoms are then assigned to one
label or the other in the ``positions_abs`` block.

Default Masses
===================================

The ionic masses that ONETEP uses by default are tabulated here, which are generally natural
abundance standard atomic weights.

===   ===   ==========   ===   ===   ==========   ===   ===   ==========
Z     El.   Mass / amu   Z     El.   Mass / amu   Z     El.   Mass / amu
===   ===   ==========   ===   ===   ==========   ===   ===   ==========
1     H     1.00794      38    Sr    87.62        75    Re    186.207
2     He    4.00260      39    Y     88.90585     76    Os    190.23
3     Li    6.941        40    Zr    91.224       77    Ir    192.217
4     Be    9.012187     41    Nb    92.90638     78    Pt    195.078
5     B     10.811       42    Mo    95.94        79    Au    196.96655
6     C     12.0107      43    Tc    98.0         80    Hg    200.59
7     N     14.00674     44    Ru    101.07       81    Tl    204.3833
8     O     15.9994      45    Rh    102.90550    82    Pb    207.2
9     F     18.99840     46    Pd    106.42       83    Bi    208.98038
10    Ne    20.1797      47    Ag    107.8682     84    Po    209.0
11    Na    22.98977     48    Cd    112.411      85    At    210.0
12    Mg    24.3050      49    In    114.818      86    Rn    222.0
13    Al    26.98154     50    Sn    118.710      87    Fr    223.0
14    Si    28.0855      51    Sb    121.760      88    Ra    226.0
15    P     30.97376     52    Te    127.60       89    Ac    227.0
16    S     32.066       53    I     126.90447    90    Th    232.0381
17    Cl    35.4527      54    Xe    131.29       91    Pa    231.03588
18    Ar    39.948       55    Cs    132.90545    92    U     238.0289
19    K     39.0983      56    Ba    137.327      93    Np    237.0
20    Ca    40.078       57    La    138.9055     94    Pu    244.0
21    Sc    44.95591     58    Ce    140.116      95    Am    243.0
22    Ti    47.867       59    Pr    140.90765    96    Cm    247.0
23    V     50.9415      60    Nd    144.24       97    Bk    247.0
24    Cr    51.9961      61    Pm    145.0        98    Cf    251.0
25    Mn    54.93805     62    Sm    150.36       99    Es    252.0
26    Fe    55.845       63    Eu    151.964      100   Fm    257.0
27    Co    58.93320     64    Gd    157.25       101   Md    258.0
28    Ni    58.6934      65    Tb    158.92534    102   No    259.0
29    Cu    63.546       66    Dy    162.50       103   Lr    262.0
30    Zn    65.39        67    Ho    164.93032    104   Rf    261.0
31    Ga    69.723       68    Er    167.26       105   Db    262.0
32    Ge    72.61        69    Tm    168.93421    106   Sg    263.0
33    As    74.92160     70    Yb    173.04       107   Bh    264.0
34    Se    78.96        71    Lu    174.967      108   Hs    265.0
35    Br    79.904       72    Hf    178.49       109   Mt    268.0
36    Kr    83.80        73    Ta    180.9479
37    Rb    85.4678      74    W     183.84
===   ===   ==========   ===   ===   ==========   ===   ===   ==========

References
===================================

.. [Skylaris2005] C.-K. Skylaris et al., J. Chem. Phys. **122**, 084119 (2005).
