Configuration and Preprocessing Data
######################################

``larcv`` is configured via ``json``.  We use `JSON for Modern C++ <https://json.nlohmann.me/>`_ to embed this into the C++ API, and the nice package `Pybind11_Json <https://github.com/pybind/pybind11_json>`_ to connect the configuration syntax to Python and dictionaries easily.


It's not expected that our users will want to delve into the C++ API to learn what configuration options are available to them - nor will we as developers want require that.  Instead, we've built a system where every configuration option in ``larcv`` has a default, that you can inspect, and therefore you can modify the default config to your liking by simply updating python dictionaries.

>>> import larcv
>>> import json
>>> json.dumps(larcv.IOManager.default_config(), indent=4)
'{
    "IOMode": 0,
    "Input": {
        "InputFiles": [],
        "ReadOnlyName": [],
        "ReadOnlyType": [],
        "UseH5CoreDriver": false
    },
    "Output": {
        "Compression": 1,
        "OutFileName": "",
        "StoreOnlyName": [],
        "StoreOnlyType": []
    },
    "Verbosity": 2
}'

The default config is a **static** member function of larcv classes.  This means that in Python, you do not need to create the object to call the static member function, you can get the default config as soon as you import ``larcv``.

Process Driver
--------------------

``larcv`` contains an internal Event-Loop style processor, ``larcv.ProcessDriver``.  You can use this to process your data sets, loop over them and run C++ alogorithms (if you need to use one of our pre-built algorithms, or you can build your own).

You can run the ProcessDriver from the command line with the ``run_processor.py`` executable script:

>>> run_processor.py -c configuration.cfg -il input0.h5 [input1.h5 ...] -ol output.h5

This functionality is meant to enable batched, single-pass processing of files where the output is preprocessed and ready to consumer in a downstream application.  If you have very large datasets and don't need everything down stream, you can trim down your output datafiles by using the ``StoreOnlyName`` and ``StoreOnlyType`` parameters in the IOManager configuration.

Note: if you are preprocessing multiple files into one output file, they will get merged automatically (and optionally can be randomized - check out the ``larcv.ProcessDriver.default_config()`` to see the parameters).  But if you just want to merge files, you can use:

>>> $ merge_larcv3_files.py --h
usage: merge_larcv3_files.py [-h] -il LARCV_FIN [LARCV_FIN ...]
                             [-ol LARCV_FOUT]

                  
ConfigBuilder
--------------------

The Configuration Builder is still in development.  But, it is meant to simplify your life and let you write configuration files, as needed, without actually writing a configuration file.  The ``ConfigBuilder`` class lets users generate most of a configuration file specifying only input/output parameters and the names of the processes they want to run, without getting bogged down in details.

Preprocessing Data
----------------------

One common pattern in machine learning, typically employed to extend the usability of a dataset, is image augmentation: random flipping, cropping, etc.  With ``larcv``, we recognize that some of these operations make sense to perfrom with the IO layer, while some operations are more reasonable to perform on an accelerator like a GPU at training time.  With a Queue IO workload, where "next" data is loaded from disk by ``larcv`` while the "current" data is being used by an application on a GPU, etc., the advantages of preprocessing with ``larcv`` can be significant.  In particular, any preprocessing steps happen on the CPU while the data on the GPU is being processed, and latency from preprocessing can be reduced or eliminated.

Preprocessed data can frequently produce new dataproducts that aren't in File and aren't meant to go to file.  ``larcv`` supports this with an IOMode that allows in-memory storage of data products until the next event is processed.  From a user perspective, any data created in memory looks like it was on disk when you go to access it again - but the reality is that no data is written to disk.
(The default write step happens when you call ``save_entry`` in ``larcv.IOManager``.)

