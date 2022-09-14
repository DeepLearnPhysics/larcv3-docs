Introduction
#############

Welcome to the documentation page for ``larcv``.

Here, we give an overview of the purpose and use cases of `larcv`, as well as some overview of required dependencies and installation instructions.

As a summary, ``larcv`` is a *storage framework* for sparse data, typically "voxelized" data (think images) but work for Point Cloud data is also in progress.  ``larcv`` was born from the MicroBooNE experiment at Fermilab, near Chicago, Illinois.  MicroBooNE is a liquid argon time projection chamber, and ``larcv`` came about to solve IO issues when storing and retrieving sparse data for machine learning applications.  The name is a combination of **L** iquid **Ar** gon **C** omputer **V** ision, originally as LArCV.


History
---------

LArCV was originally built in C++, using the ``ROOT`` framework for IO and data manipulation.  As machine learning applications moved to bigger systems, we adapted ``larcv`` to use more industry-standard tools (HDF5 instead of ROOT).  We also adapted the data products storable here to be more genericlly useful beyond the MicroBooNE experiment.  To not inhibit experiments already using a verion of larcv at the time, we forked the repository and allowed ``larcv2`` (an update to LArCV) and ``larcv3`` to coexist.  You've found your way to the documentation for ``larcv3`` - apart from this introduction section, anywhere in these docs where you find ``larcv``, we're refering to ``larcv3``.  The repositories for each are:

``larcv2``: 

`https://github.com/DeepLearnPhysics/larcv2 <https://github.com/DeepLearnPhysics/larcv2>`_

``larcv3``:

`https://github.com/DeepLearnPhysics/larcv3 <https://github.com/DeepLearnPhysics/larcv3>`_


Use Cases
-----------

``larcv`` is open source and you can use it, modify it, etc., for whatever purpose is needed.  If you're thinking about ``larcv`` for your project, here are some good criteria your project should meet for ``larcv`` to be useful for you: 

* Your application is either in python or C++.  ``larcv`` isn't supported in other languages with no intention to branch out at this time.

* Your application needs to use sparse input data, probably for machine learning.  Right now that means sparse data in a regular, voxelized space, eventually that will include points in more general space, or;

* Your application uses dense but very regular data.  Meaning, the image size for each iteration of your algorithm is the same everytime. ``larcv`` does not have restrictions on image size, but dense data is optimized for readback when the images are regular.

* You need to improve the speed of IO in your application at scale.  ``larcv`` has native scalability for IO readback at the scale of the largest super computers, up to 30,000+ MPI Ranks.


If this suits your needs, ``larcv`` might be useful to you.  If it seems *almost* useful but missing some features, please feel free to open an issue or feature request on our `github page <https://github.com/DeepLearnPhysics/larcv3>`_.

.. toctree::
   :maxdepth: 1
   :caption: Contents:
   :name: mastertoc

   install
   quickstart
   datamodel
   performance
   visualization
   API

.. Indices and tables
.. ==================

.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`

