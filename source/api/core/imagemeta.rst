ImageMeta
################

`larcv` is built on the idea that sparse data exists in a real space: active pixels in a detector, for example, or thresholded points in an image.  The data, however, have a relationship to the full space and to all the other auxilliary data products that come with them.  To simplify the mapping between data products in `larcv`, we implement the concept of `ImageMeta`.


With the exception of the `particle` data product, each of the data products that `larcv` can serialize has an associated vector of projection IDs which can serve as a multipurpose enumeration of detectors: it can be the multiple wire-planes of a LArTPC, or several readout regions in a segmented detector.  It is up the creator of the data to decide.  To describe the properties of the volume indicated by the projection ID, an ImageMeta object is stored with each projection that describes the physical volume in terms of number of voxels, voxel size, and origin across all dimensions.  

The ImageMeta objects also provide a convenient interface for raveling/unraveling a multi-dimension index, identical to the `numpy` scheme of mapping a multi-index to a single flattened array index.  The ImageMeta interface is shared across all data products that have a spatial relation, ensuring consistent development patterns.  The ImageMeta objects also provide the absolute positioning of all objects, enabling full images to be built from objects that represent only a subset of a detector, potentially.

Below, we include the API for `ImageMeta2D`, but please note that ImageMeta is available as `ImageMeta1D`, `ImageMeta2D`, `ImageMeta3D`, and `ImageMeta4D`.  The API is the same in all cases (in fact in C++ these are just different template specializations.)

.. automodule:: larcv
    :noindex:

.. autoclass:: ImageMeta2D
    :members:

