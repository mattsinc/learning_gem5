
:authors: Jason Lowe-Power

.. _building-chapter:

--------------
Building gem5
--------------

This chapter covers the details of how to set up a gem5 developmment environment and build gem5.

.. todo::

    Add a pointer to the gem5 docker image.
    In fact, we may want to have a docker image for each section.

.. _building-requirements-section:

Requirements for gem5
~~~~~~~~~~~~~~~~~~~~~

See `gem5 requirements`_ for more details.

.. _gem5 requirements: http://gem5.org/Compiling_M5#Required_Software

#. hg (Mercurial_):
    The gem5 project uses Mercurial_ for version control.
    Mercurial_ is a distributed version control system (like git).
    It uses simple commands that should be familiar to svn users as well.
    More information about Mercurial_ can be found by following the link.
    Mercurial should be installed by default on most platforms.
    However, to install Mercurial in Ubuntu use

    .. code-block:: sh

        sudo apt-get install mercurial

#. gcc 4.6+
    You may need to use environment variables to point to a non-default version of gcc.
    For CSL machines, you can add the following to your `.bashrc.local`, assuming you're using bash as your shell.

    .. code-block:: sh

        export PATH=/s/gcc-4.7.3/bin:$PATH
        export LD_LIBRARY_PATH=/s/gcc-4.7.3/lib64:$LD_LIBRARY_PATH

    On Ubuntu, you can install a development environment with

    .. code-block:: sh

        sudo apt-get install build-essential

#. SCons_
    gem5 uses SCons as its build environment.
    SCons is like make on steroids and uses Python scripts for all aspects of the build process.
    This allows for a very flexible (if slow) build system.

    To get SCons on Ubuntu use

    .. code-block:: sh

        sudo apt-get install scons

#. Python 2.5+
    gem5 relies on the Python development libraries.
    On CSL machines, you may experience errors with the default Python version (2.5).
    To use version 2.7 you can add the following to your `.bashrc.local`:

    .. code-block:: sh

        export PATH=/s/python-2.7.3/bin:$PATH
        export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/s/python-2.7.3/lib
        export LIBRARY_PATH=/s/python-2.7.3/lib:$LIBRARY_PATH

    To install these on Ubuntu use

    .. code-block:: sh

        sudo apt-get install python-dev

#. SWIG_ 2.0.4+
    SWIG_ is a set of scripts and libraries that wraps `C++` objects and exports them to scripting languages, like Python.
    gem5 uses SWIG to export `C++` SimObjects to the Python configuration files.

    To install SWIG on Ubuntu use

    .. code-block:: sh

        sudo apt-get install swig

    On CSL machines, you can use swig found in ``/s``:

    .. code-block:: sh

        export PATH=/s/swig-2.0.6/bin/:$PATH

    You may have to install SWIG manually.
    In that case, you can download the source from http://www.swig.org/download.html.
    Version 2.0.4 is known to work with gem5.
    Then, unpack, build, and install:

    .. code-block:: sh

        tar -xvzf swig-2.0.4.tar.gz
        ./configure --prefix=<PATH INSTALL SWIG. e.g., ~/local>
        make && make install

#. protobuf_ 2.1+
    "Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data."
    In gem5, the protobuf_ library is used for trace generation and playback.
    protobuf_ is not a required package, unless you plan on using it for trace generation and playback.

    .. code-block:: sh

        sudo apt-get install libprotobuf-dev python-protobuf protobuf-compiler libgoogle-perftools-dev

.. _Mercurial: http://mercurial.selenic.com/

.. _SCons: http://www.scons.org/

.. _SWIG: http://www.swig.org/

.. _protobuf: https://developers.google.com/protocol-buffers/

Getting the code
~~~~~~~~~~~~~~~~

Change directories to where you want to download the gem5 source.
Then, to clone the repository, use the ``hg clone`` command.

.. code-block:: sh

  hg clone http://repo.gem5.org/gem5

You can now change directories to ``gem5`` which contains all of the gem5 code.

.. Removed for now since we got rid of gem5-stable, but it may come back later.
    .. sidebar:: gem5 repositories

        There are two main gem5 repositories found on repo.gem5.org, *gem5*, and *gem5-stable*.
        gem5 is the main development repository, which is updated very frequently (a few times per week).
        This repository has all of the latest bugfixes and features.
        However, there are often bugs introduced and changes to APIs.
        gem5-stable, is released once every few months and pulls in most of the changes to gem5 in that time.
        It's more stable than the gem5 repository, but there still may be bugs.

        If you find a bug in gem5-stable, or something isn't working correctly, be sure to try gem5 before submitting a bug report.
        The problem may already be fixed.

Your first gem5 build
~~~~~~~~~~~~~~~~~~~~~~~
Let's start by building a basic x86 system.
Currently, you must compile gem5 separately for every ISA that you want to simulate.
Additionally, if using :ref:`ruby-intro-chapter`, you have to have separate compilations for every cache coherence protocol.

To build gem5, we will use SCons.
SCons uses the SConstruct file (``gem5/SConstruct``) to set up a number of variables and then uses the SConscript file in every subdirectory to find and compile all of the gem5 source.

SCons automatically creates a ``gem5/build`` directory when first executed.
In this directory you'll find the files generated by SCons, the compiler, etc.
There will be a separate directory for each set of options (ISA and cache coherence protocol) that you use to compile gem5.

There are a number of default compilations options in the ``build_opts`` directory.
These files specify the parameters passed to SCons when initially building gem5.
We'll use the X86 defaults and specify that we want to compile all of the CPU models.
You can look at the file ``build_opts/X86`` to see the default values for the Scons options.
You can also specify these options on the command line to override any default.

.. code-block:: sh

    scons build/X86/gem5.opt -j9

.. sidebar:: gem5 binary types

    The SCons scripts in gem5 currently have 5 different binaries you can build for gem5: `debug`, `opt`, `fast`, `prof`, and `perf`.
    These names are mostly self-explanatory, but detailed below.

    debug
        Built with no optimizations and debug symbols.
        This binary is useful when using a debugger to debug if the variables you need to view are optimized out in the `opt` version of gem5.
        Running with `debug` is slow compared to the other binaries.

    opt
        This binary is build with most optimizations on (e.g., -O3), but with debug symbols included.
        This binary is much faster than `debug`, but still contains enough debug information to be able to debug most problems.

    fast
        Built with all optimizations on (including link-time optimizations on supported platforms) and with no debug symbols.
        Additionally, any asserts are removed, but panics and fatals are still included.
        `fast` is the highest performing binary, and is much smaller than `opt`.
        However, `fast` is only appropriate when you feel that it is unlikely your code has major bugs.

    prof and perf
        These two binaries are build for profiling gem5.
        `prof` includes profiling information for the GNU profiler (gprof), and `perf` includes profiling information for the Google performance tools (gperftools).


The main argument passed to SCons is what you want to build, `build/X86/gem5.opt`.
In this case, we are building gem5.opt (an optimized binary with debug symbols).
We want to build gem5 in the directory build/X86.
Since this directory currently doesn't exist, SCons will look in ``build_opts`` to find the default parameters for X86.
(Note: I'm using -j9 here to execute the build on 9 of my 8 cores on my machine.
You should choose an appropriate number for your machine, usually cores+1.)

The output should look something like below:

::

  Checking for C header file Python.h... yes
  Checking for C library pthread... yes
  Checking for C library dl... yes
  Checking for C library util... yes
  Checking for C library m... yes
  Checking for C library python2.7... yes
  Checking for accept(0,0,0) in C++ library None... yes
  Checking for zlibVersion() in C++ library z... yes
  Checking for GOOGLE_PROTOBUF_VERIFY_VERSION in C++ library protobuf... yes
  Checking for clock_nanosleep(0,0,NULL,NULL) in C library None... yes
  Checking for timer_create(CLOCK_MONOTONIC, NULL, NULL) in C library None... no
  Checking for timer_create(CLOCK_MONOTONIC, NULL, NULL) in C library rt... yes
  Checking for C library tcmalloc... yes
  Checking for backtrace_symbols_fd((void*)0, 0, 0) in C library None... yes
  Checking for C header file fenv.h... yes
  Checking for C header file linux/kvm.h... yes
  Checking size of struct kvm_xsave ... yes
  Checking for member exclude_host in struct perf_event_attr...yes
  Building in /local.chinook/gem5/gem5-tutorial/gem5/build/X86
  Variables file /local.chinook/gem5/gem5-tutorial/gem5/build/variables/X86 not found,
    using defaults in /local.chinook/gem5/gem5-tutorial/gem5/build_opts/X86
  scons: done reading SConscript files.
  scons: Building targets ...
   [ISA DESC] X86/arch/x86/isa/main.isa -> generated/inc.d
   [NEW DEPS] X86/arch/x86/generated/inc.d -> x86-deps
   [ENVIRONS] x86-deps -> x86-environs
   [     CXX] X86/sim/main.cc -> .o
   ....
   .... <lots of output>
   ....
   [   SHCXX] nomali/lib/mali_midgard.cc -> .os
   [   SHCXX] nomali/lib/mali_t6xx.cc -> .os
   [   SHCXX] nomali/lib/mali_t7xx.cc -> .os
   [      AR]  -> drampower/libdrampower.a
   [   SHCXX] nomali/lib/addrspace.cc -> .os
   [   SHCXX] nomali/lib/mmu.cc -> .os
   [  RANLIB]  -> drampower/libdrampower.a
   [   SHCXX] nomali/lib/nomali_api.cc -> .os
   [      AR]  -> nomali/libnomali.a
   [  RANLIB]  -> nomali/libnomali.a
   [     CXX] X86/base/date.cc -> .o
   [    LINK]  -> X86/gem5.opt
  scons: done building targets.

When compilation is finished you should have a working gem5 executable at ``build/X86/gem5.opt``.
The compilation can take a very long time, often 15 minutes or more, especially if you are compiling on a remote file system like AFS or NFS.


Common errors
~~~~~~~~~~~~~~

Wrong gcc version
==================

::

    Error: gcc version 4.6 or newer required.
           Installed version: 4.4.7

Update your environment variables to point to the right gcc version, or install a more up to date version of gcc.
See :ref:`building-requirements-section`.

Wrong SWIG version
===================

::

    Error: SWIG version 2.0.4 or newer required.
           Installed version: 1.3.40

Update your environment variables to point to the right SWIG version, or install a more up to date version of SWIG.
See :ref:`building-requirements-section`.

Python in a non-default location
================================

If you use a non-default version of Python, (e.g., version 2.7 when 2.5 is your default), there may be problems when using SCons to build gem5.
RHEL6 version of SCons uses a hardcoded location for Python, which causes the issue.
gem5 often builds successfully in this case, but may not be able to run.
Below is one possible error you may see when you run gem5.

::

    Traceback (most recent call last):
      File "........../gem5-stable/src/python/importer.py", line 93, in <module>
        sys.meta_path.append(importer)
    TypeError: 'dict' object is not callable

To fix this, you can force SCons to use your environment's Python version by running ``python `which scons` build/X86/gem5.opt`` instead of ``scons build/X86/gem5.opt``.
More information on this can be found on the gem5 wiki about non-default Python locations: `Using a non-default Python installation <http://www.gem5.org/Using_a_non-default_Python_installation>`_.

M4 macro processor not installed
================================

If the M4 macro processor isn't installed you'll see an error similar to this:

::

    ...
    Checking for member exclude_host in struct perf_event_attr...yes
    Error: Can't find version of M4 macro processor.  Please install M4 and try again.

Just installing the M4 macro package may not solve this issue.
You may nee to also install all of the ``autoconf`` tools.
On Ubuntu, you can use the following command.

.. code-block:: sh

    sudo apt-get install automake
