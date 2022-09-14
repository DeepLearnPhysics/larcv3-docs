Future Projects
#####################

``larcv`` is in a stable, mature state and the API is not expect to alter significantly in the future.  There are, however, some projects that we're considering when they are needed, and they're listed here in case you want to see what's coming, request more features, or suggest we prioritize something in particular.  Please, open a github issue / feature request to tell us of your thoughts, if you want to see things move faster.

* Standardization of API calls for ``Event*`` objects, as well as for container type objects (``VoxelSet``, ``VoxelSetArray`` and their derived classes).  Additionally, standardized way to access ``meta`` objects for classes that contain a ``meta`` object.

* Templating of IO calls for ``Event*`` objects to enable easier maintenance and eventual upgrading of serialization layers.

* Improved IO performance for large batch data reading on a single rank.  Currently, this latency can be high, and it is difficult to improve because HDF5 is not thread safe.  Any internal locking requires careful work with the MPI layer.

* Introduction of Point Cloud data layers.  Will be added as soon as there is a use case or user request.