==============
Developer Area
==============

:Author: Jacek Dziedzic, University of Southampton
:Author: James C. Womack, University of Southampton
:Author: Nicholas Hine, University of Warwick

.. _dev_code_quality:

Advice for Contributors
=======================


.. _dev_pre_pull:

Pre-pull-request checks
-----------------------

The ONETEP source code is distributed with a number of useful scripts for
checking some aspects of the correctness of source code. Before creating a
pull request, you should ensure that these tests all pass for the branch you
want to merge in your private fork:

  1. **Check module dependencies** and a few other common issues. Run
     ::

       make checkdeps

     or, equivalently,

     ::

       ./utils/check_dependencies

     in the root directory of the repository. If any errors are found, fix them.

  2. **Check whitespace**. Run
     ::

       ./utils/check_whitespace

     in the root directory of the repository. This script can automatically fix
     whitespace issues -- see the output of ``./utils/check_whitespace -h`` for
     details.

  3. **Check use clauses**. Run
     ::

       ./utils/check_use_clauses src/yourmodule_mod.F90

     in the root directory of the repository, replacing ``yourmodule`` with the
     module you worked on. If there was more than one, do this for every module
     separately. This script checks for unnecessary
     use clauses and will help point them out. You are only responsible for the
     use clauses in the code you touched.

  4. You must also run the full QC test suite before creating a pull request.
     Instructions for that can be found here: :ref:`starting_compiling`.

|

Key points about running the QC test suite before creating a pull request:
  * Never run the QC tests in the actual repository. Create a copy of your
    ONETEP installation directory (e.g. ``cp -a onetep_jd onetep_jd_qc``) and
    run the tests in the directory of the copy. Much of the stuff in the install
    is not needed to run the tests (e.g. all of ``src/``), but it's just less
    hassle to copy the entire thing than pick out the necessary subdirectories.

    Why not run the QC tests in the actual repository? Because they produce
    outputs, which can become messy to clean up back to pristine state,
    polluting your repository with spurious changes. This becomes more
    problematic because symbolic links are used in the QC test suite. Trying
    to update your repository via ``git pull`` or ``git fetch`` may be
    tricky when you have unfinished or improperly cleaned up QC tests. Just
    don't run them straight in the repository.

   * You should run the test suite with ONETEP compiled with a recent version of
     a widely used Fortran compiler, e.g. GNU Fortran, Intel Fortran Classic,
     Cray Fortran.

   * The tests should be run in a parallel environment (ONETEP must be compiled
     with MPI support), ideally also making use of OpenMP threading (set the
     environment variable ``OMP_NUM_THREADS`` to a value >1, or use the ``-t``
     argument to ``onetep_launcher``).

   * No tests should fail, though warnings are acceptable.

If tests fail, please investigate this further before creating your pull request:
  * Check out the master branch from the official repository (ensuring that it is
    fully updated by fetching from it and merging) and run the failing tests on
    this.
  * If the relevant tests do not fail upstream (i.e. for the master branch on
    the official repository), you should fix the code in your fork before
    creating your pull request.
  * If the relevant tests also fail upstream, please check the
    list of issues in the official repository (
    https://bitbucket.org/onetep/onetep/issues, soon to move to
    https://github.com/onetep-devel/onetep/issues): there should be an open
    issue with relevant information. If such an issue has not been reported,
    please create a new issue explaining which tests are failing and any
    details about how the failures occur.

If the tests are not failing upstream, the problem is likely to be part of the
changes you have implemented and you will be the best person to find it and fix
it. If you need help, and want to discuss the issue with other developers, you
may create an issue in your fork (not in the official repository) and draw
their attention to it via email or on the ONETEP development Slack at
https://onetep-devel.slack.com/.

If your tests fail in a way in which the official master branch is known to
fail, your pull request may be acceptable -- you can go ahead creating the
pull request, but make sure that the text of the pull request includes a list
of the tests that fail and why you think that this is not a problem.

Ensure that the ONETEP version number in your
``src/onetep.F90`` file is updated appropriately (see
:ref:`dev_version_numbering` for a description
of ONETEP's version numbering system). If multiple pull requests are open, or
it takes time to merge your pull request, it may be necessary to update the
version number again during the pull request code review. This is to
ensure that the version number increments linearly in the official
repository's ``master`` branch.

Read the next subsection on style conventions to ensure your code adheres to
them.


.. _dev_style_conventions:

Style conventions
-----------------

In order to keep the ONETEP codebase relatively consistent and readable,
we have adopted the following conventions:

* Each line of code should be 80 columns or fewer in length, as this makes
  comparison of two versions of code side-by-side easier (some older code does
  not adhere to this, but all new code should).
* If a line exceeds 80 columns in length, use continuation characters (``&``)
  to break it into chunks of <=80 characters (remember to include an extra
  ``&`` at the beginning of the following line when breaking in the middle
  of a string literal). Do not add ``&`` in the next line otherwise,
  it's unnecessary.
* Always use spaces for indentation, never tabs.
* The blocks in the ``do`` loop, ``select case`` (also ``select type``) and
  ``if`` constructs should be indented by **3 spaces**.
* The contents of subroutines and functions should be indented by **2 spaces**.
  Use further 2 spaces for internals.
* Line continuations should be indented by **5 spaces** for continued lines.
* Ensure there is no trailing whitespace before you commit (you can use
  the ``./utils/check_whitespace`` script described in :ref:`dev_pre_pull`
  to check this).
* In general, adding or removing blank lines should be avoided in core modules,
  as these changes will appear in the commit history.


.. _dev_new_functionality:

New functionality
-----------------

If you add new procedures or significantly change existing procedures,
**you must create or update the documentation in the source code**.
Examples of how to document procedures can be found in the template module
``template_mod.F90`` in the ``doc`` directory. The key components of the
documentation of procedures are:

* A human-readable description of the new functionality.
* A list of the arguments and a description of their meaning.
* The author(s) and a changelog describing significant modifications.

When adding new functionality which does not fit into other modules, it may
be necessary to create a new source file containing a new module. Note that
procedures and variables should always be encapsulated in modules, not 'bare'
in a source file.

Before creating a new module, you should consider carefully whether your
new functionality fits within the framework of an existing module, or is
generic enough to be part of a multi-purpose module, such as `utils` or
``services``. If a new module is needed to encapsulate some new functionality,
then you should follow the following guidelines:

* Give your module a name which indicates the functionality it contains.
  If unsure, consult a more experienced developer to discuss an appropriate name.
* The filename for the module should have the form ``<module_name>_mod.F90``,
  where ``<module_name>`` is the name you have given the module.
* By default, variables and procedures in your module should be private (i.e.
  they should have the ``private`` attribute).
* Global module-wide variables (private or public variables declared at the
  level of the module itself rather than within its routines) constitute
  "hidden state", which tends to make the behaviour of a routine undesirably
  dependent on more than just the arguments it is called with. Sometimes these
  are unavoidable, and there are instances of them in the code. However, they
  should be minimised as much as possible. Think carefully before declaring any
  module-level global variables. More experienced developers may be able to
  suggest ways to encapsulate data inside arguments to routines such that they
  do not constitute "hidden state".
* Variables and procedures which do have to be public (accessible outside the
  module) should be explicitly specified (i.e. they should have the ``public`` a
  ttribute).
* In general, public variable and procedure names should be prepended by a
  standard prefix (typically the module name, or a shortened version of the
  name).

It is recommended that you make a copy of ``./doc/template_mod.F90`` and use
this as a starting point for your new module, as this will make following the
above guidelines easier.

.. _dev_version_numbering:

Version numbering
-----------------

There are three parts to the version number, both for development versions and
release versions. The first version number is only very rarely incremented by a
collective decision of the main authors of the code (ODG). New major versions
are released around every 6-12 months and are indicated by incrementing the
second number in the full version number (e.g. "2" in "4.2").

The major version number (second number in full version number) indicates
whether the associated source code is a release version or a development
version:

* Release versions (which are distributed to users) have an *even*
  second number.
* Development versions (which are under active development) have an *odd*
  second number.

Within a series with the same second version number, successive versions
(indicated by the third number in the full version number) should be compilable
and complete with respect to a given new feature.

For minor changes in development versions (e.g. a bugfix or minor change to
existing code), we no longer increment the fourth number (which is now absent
altogether) to avoid merge conflicts when this is done by multiple people.
The script ``utils/embed_version_info_in_banner`` ensures that pertinent
details of the local repository (branch, remote, last commit ID, list of
locally modified files) are included in the ONETEP banner during compilation,
but they do not go into the repository. Major changes (e.g. a new module or
overhaul of existing functionality) should increment the third number.

Bugfixes to a release version (merged into the corresponding release branch on
the official repository, e.g. `academic_release_v5.0`) should increment the
last (third) number in the full release version number.

**At any given time, there is a development version and a release version
differing in their second version number by 1.**

  * 4.0.0   <-- first release of v4
  * 4.0.1   <-- bugfix to v4 in git branch for release
  * 4.1.0   <-- first development version of v4 (initially same as 4.0.0)
  * 4.1.0   <-- minor development work (changes will be summarised in banner)
  * 4.1.1   <-- significant development work
  * 4.2 RC3 <-- release candidate 3 for v4.2
  * 4.2.0   <-- next release version
  * 4.3.0   <-- next development version (initially same as 4.2.0).
  * 5.0.0   <-- first release of v5


.. _dev_preventing:

Preventing accidental pushes to the official repository
=======================================================

* GitHub users in the *Owner* role of the ONETEP repository have write access
  to the official repository.
* *Owners* may want to take steps to avoid accidentally pushing work to the
  official repository if they have added this as a remote to their private fork.
* This can be achieved by setting the push address for the remote to an
  unresolvable URL, e.g.:
  ::

    git remote set-url --push github_official DISABLE


.. _dev_history:

History
=======

The earliest versions of the code, dating back to before 2005, were committed
to a revision control system based on CVS. These files are still available on
the TCM filesystem at /u/fs1/onestep/CVS_REPOSITORY. In around 2009 we moved
to Subversion, using a repository still hosted on the TCM filesystem in
Cambridge. The SVN history was migrated to Bitbucket, and subsequently to
GitHub (2023) and can still be browsed within the current Git repository
(though attribution to authors is often not correctly recorded).

In June 2018, the ONETEP project was migrated from a Subversion repository to a
Git repository. The hosting of source code was simultaneously moved
from cvs.tcm.phy.cam.ac.uk to Bitbucket.

In July 2023, the ONETEP project was migrated from Bitbucket to GitHub.
