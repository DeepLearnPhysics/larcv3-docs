Installation
###############

Installing ``larcv`` is not meant to be complicated.  You need two required and one optional non-python package;  You need a C++ compiler that is compatible with the python you want to use;  You need a couple packages available on PyPi; After that, it should not be hard to install.

Required Dependencies:
--------------------------------

* HDF5 - Available through package managers on your system;  Not too challenging to build from source if needed.  From some package managers, you may need the "dev" version.

* Cmake - Required for building the C++ layers, not needed at runtime.  Available through ``pip`` and also from package managers.

* (Optional) MPI - Needed to run ``larcv`` on more than one process at a time.  Optionally, you can use HDF5 with MPI-IO enabled, but that requires a build-time configuration.

* numpy - ``larcv`` is tightly integrated with ``numpy`` through pybind11.

* scikit-build - used to run cmake and build ``larcv`` into a python package.  Not used at runtime.

* h5py - needed if you want to use some of the file management tools, for merging larcv .h5 files.

* (Optional) PyTest - if you want to test your installation, ``larcv`` has a full test suite.

Installing From Source
--------------------------------

``larcv`` can be installed from source or via PyPi.  To install from source, please do:

>>> git clone https://github.com/DeepLearnPhysics/larcv3.git

If installing from source, please note that you need to do:

>>> git submodule update --init

once from somewhere in the repository.  Several C++ dependencies are included directly as git submodules (nlohmann_json, pybind11, pybind11_json) and aren't downloaded by default.

Next, build the code with

>>> python setup.py build -j 32

With CMake, it's recommended to include the ``-j 32`` (or another number) to enable the parallel build.

Installing From PyPI
--------------------------------

``larcv`` can be installed from PyPI as well.  To do so, just do ``pip install larcv`` as expected.  Your build may break if you don't have hdf5 installed (including headers).  A few tips for your installation:

* The C++ build component can take awhile.  It is single-core by default (otherwise, it breaks some CI systems) but you can enable a parallel build like so:

>>> MAKE_FLAGS="-j 32" pip install larcv

Where the number after ``-j`` is the build parallelization.

* Use larcv version at least 3.4.2.1 for PyPI builds.  A few other builds are not as stable.

Why no manylinux build?
************************

We haven't deployed a manylinux build yet but it's in the works.  One challenge is how to handle the HDF5 library dependencies.

Some other build options
--------------------------------

There are a few other options you can use to configure your build of ``larcv``, typically set as env variables at build time.

* ``MAKE_FLAGS="-j 32"``, where 32 can be replaced with any number.  This parallelizes the build, recommended if you are building ``larcv`` yourself.

* ``LARCV_WITH_MPI=1`` enables MPI HDF5 levels.  You can still run your workloads without the HDF5-MPI-IO layer, it will still work, but for some workloads you may see a performance improvement building with this flag.

* ``LARCV_WITH_OPENMP=1`` enables OpenMP optimizations for some preprocessing apps.  Unless you are seeing bottle necks in preprocessing, you probably don't need this.

* ``LARCV_WITHOUT_PYBIND=1`` will disable python bindings in the build.  This is most useful when you need to connect your C++ simulation code to larcv, and the simulation is not friendly to python code.