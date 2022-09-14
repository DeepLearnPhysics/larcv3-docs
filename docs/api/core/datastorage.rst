Data Storage
################

``larcv`` Data storage is built upon HDF5, including ability to use MPI-IO if needed, 

Each ``larcv`` file is divided into two H5Groups: **Events** and **Data**.  Within **Events** there is only one table, **event_id**, which enumerates the global bookkeeping properties of each event in the file.  Events may be stored in any order, and are tagged through a run/subrun/event indexing scheme.  A run is usually an index that is constant through one data-taking or simulation-generating unit of a dataset, where detector conditions are constant, as dictated by a particular experiment.  A subrun frequently indicates a sub-file within a run, and events indicate an entry index within a file.  Though ``larcv`` is based on real-world situations with run/subrun/event indexing, these are purely informational objects in ``larcv``.

All data products are stored in a product/producer pair: there is no specific limit on the number of a particular data product stored, but it must have a unique producer name.  Further, the data product for each producer is stored for every event.  To enable a complete implementation of sparse and dense tensors, as well as sparse cluster sets and particle arrays, ``larcv`` uses a mix of inheritance and dimensional templating to have a minimal code base for both serialization/deserialization and python bindings.

All data storage access uses ``Event*`` objects, such as ``larcv.EventParticle`` or ``larcv.EventSparseTensor3D``.  You access these types via the IOManager, which registers data access on the fly to ensure proper serialization is done when you finish modifying.  In C++:

>>> // Access with const to get immutable, read-only data:
>>> auto const & ev_input  = mgr.template get_data<dataproduct_in>(producer);
>>>
>>> // Access without constant to get write-access:
>>> auto       & ev_output = mgr.template get_data<dataproduct_out>(output_producer);
>>>
>>> /*
>>>  Do your ev_output manipulation here
>>> */
>>>
>>> // IO Manager will automatically save if output is enabled


Or, in Python:

>>> event_tensor2d = io_manager.get_data(dataproduct_string, producer_name)
