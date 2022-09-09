Data Model
##################

Here, we describe the data model driving the serialization and processing of `larcv`, and note some of it's history.  If and when new data products are added, we will add a description of them here and their motivation.

To enable efficient IO within `larcv`, we have designed a data model where users will format their data to fit into several abstract, optimized data products which can then be efficiently and automatically serialized or de-serialized.  With `HDF5`  as a backend for IO, we take advantage of the high performance, open-source and cross-platform file formats.  We build several data types on top of this to accommodate the unique needs of sparse, high energy physics data.  Additionally, `HDF5` enables `larcv` to handle sparse IO at scale with MPI-IO as needed.

Particle physics data is typically multi-channeled with several sources of data per single observed (or simulated) interaction.  The atomic unit of a particle physics data-set is often called an **event** and represents one self-contained observation (or simulation) of a detector.  An event can contain images from an imaging detector, segmentation labels from a simulation, and multiple projections of a single detector's readout.  Additionally, simulated events typically contain a list of *particle* objects representing the trajectories of fundamental particles, their energies, and their hierarchy of which particles created which other particles.  It is a fundamental design principle of `larcv` to enable efficient storage and use of these pieces.

The core data objects of `larcv` are as follows

* **Tensors (1D, 2D, 3D or 4D)** - A tensor is a dense, memory-contiguous object that can be interpreted as a multi-dimensional array.  For better integration into the Python ecosystem, the indexing scheme adopted by `larcv` to map from contiguous memory location to N-dimensional index is deliberately identical to the scheme used by `numpy`.  Once deserialized from disk, a Tensor in `larcv` can be yielded to `numpy` with zero data copy.  Examples of Tensor2D data are shown below.  

.. image:: figures/data_products/cosmic_tagger_wire_image.pdf
    :width: 45%
    :alt: Examples of 2D data, from the CosmicTagger dataset.  Shown is a  dense representation of simulated input data, with an artificial blue background.  


.. image:: figures/data_products/cosmic_tagger_seg_label_0.pdf
    :width: 45%
    :alt: Right is the sparse representation of the corresponding segmentation labels.

* **SparseTensor (2D or 3D)** - A sparse tensor is a zero-suppressed implementation of a Tensor object, where the key data encoded is a `Voxel` object consisting of a (value, index) pair.  The index represents the index of this voxel if it were stored in a contiguous-memory tensor, while the value is self-explanatory. Sparse tensors are zero-suppressed in concept, but it is permissible to deliberately store a 0.0 value if that is needed by an application.  SparseTensors can represent the intrinsic data of an experiment, or derived objects such as sparse segmentation masks.  SparseTensor data is shown in Figure~\ref{fig:CT_wire_segmentation}.

* **SparseCluster (2D or 3D)** - A sparse cluster is an extension of a sparse tensor where each voxel is effectively given a third property, an index of a `cluster` to which it belongs.  In particular, this is targeting instance segmentation tasks where multiple masks are part of the labeled data set for any given image.  SparseCluster data is shown in Figures~\ref{fig:dune2d_data} and \ref{fig:dune3d_data}.

* **Particle** - A particle represents high-level simulated or reconstructed information about an Event as a whole, or a list of particles can represent the detailed information about every particle tracked in an event.  Particle objects contain, for example, energy, vertex, and momentum information.  They also may be used for entire-event classification storage.  Particles can store hierarchy information, and can also be used to create hierarchies of associated objects (SparseClusters, Bounding Boxes).

* **Bounding Box  (2D or 3D)** - A rectangular object that dictates the extent of an object of interest in a detector.  Similar to the bounding boxes of object detection work in computer vision \cite{faster-rcnn}, though extended with rotation matrices to enable a broad suite of use cases (lines, rectangles, at angles, etc.).  Bounding box example data is show in Figures~\ref{fig:dune2d_data} and \ref{fig:dune3d_data}.

* **ImageMeta (1D, 2D, 3D or 4D)** - A meta class describes the voxelization of an N-dimensional space, including the location of the origin, size, and number of voxels per dimension.  While not a dataproduct in and of itself, the ImageMeta class provides the context for the spatial dataproducts and therefore also defines how they interact.

\begin{figure}[ht!]
    \centering
    \includegraphics[width=0.45\columnwidth]{figures/data_products/dune2d_evd_wires_and_bboxes.pdf}
    \includegraphics[width=0.45\columnwidth]{figures/data_products/dune2d_evd_clusters.pdf}
    \caption{Examples of 2D data, from the DUNE2D dataset (described below).  Left is a dense representation of simulated data, with an artificial blue background, overlaid with instance bounding boxes.  Right is the SparseCluster representation of the corresponding instance segmentation labels.}
    \label{fig:dune2d_data}
\end{figure}

\begin{figure}[ht!]
    \centering
    \includegraphics[angle=-90,origin=c,width=0.45\columnwidth]{figures/data_products/dune3d_voxels.png}
    \includegraphics[angle=90,origin=c,width=0.45\columnwidth]{figures/data_products/dune3d_clusters_bboxes.png}
    \caption{Examples of 3D data, from the DUNE3D dataset (described below).  Left is a dense representation of simulated data in 3D.  Right is the SparseCluster representation of the corresponding instance segmentation labels, shown with instance bounding boxes overlaid.}
    \label{fig:dune3d_data}
\end{figure}

With the exception of the Particle data product, each of the data products has an associated vector of ``projection IDs'' which can serve as a multipurpose enumeration of detectors: it can be the multiple wire-planes of a LArTPC, or several readout regions in a segmented detector.  It is up the creator of the data to decide.  To describe the properties of the volume indicated by the projection ID, an ImageMeta object is stored with each projection that describes the physical volume in terms of number of voxels, voxel size, and origin across all dimensions.  The ImageMeta objects also provide a convenient interface for raveling/unraveling a multi-dimension index, identical to the \texttt{NumPy} scheme of mapping a multi-index to a single flattened array index.  The ImageMeta interface is shared across all data products that have a spatial relation, ensuring consistent development patterns.  The ImageMeta objects also provide the absolute positioning of all objects, enabling full images to be built from objects that represent only a subset of a detector, potentially.

\subsection{Serialization Methods}

Each larcv file is divided into two H5Groups: {\bf Events/} and {\bf Data/}.  Within {\bf Events/} there is only one table, {\bf event\_id}, which enumerates the global bookkeeping properties of each event in the file.  Events may be stored in any order, and are tagged through a run/subrun/event indexing scheme.  A run is usually an index that is constant through one data-taking or simulation-generating unit of a dataset, where detector conditions are constant, as dictated by a particular experiment.  A subrun frequently indicates a sub-file within a run, and events indicate an entry index within a file.  Though \larcv is based on real-world situations with run/subrun/event indexing, these are purely informational objects in \larcv.

All data products are stored in a product/producer pair: there is no particular limit on the number of a particular data product stored, but it must have a unique producer name.  Further, the data product for each producer is stored for every event.  To enable a complete implementation of sparse and dense tensors, as well as sparse cluster sets and particle arrays, \larcv uses a mix of inheritance and dimensional templating to have a minimal code base for both serialization/deserialization and python bindings.  Python binding implementation is done via pybind11 \cite{pybind11}, while configuration is done via the \texttt{nlomann\_json} package \cite{nlohmann, json}.

% Each dataproduct in \larcv is initially stored in local memory, with buffers flushed at the transition between event indexes.  In some cases, such as on-the-fly pre-processing, it is advantageous to allow creation of data products in memory without serialization to disk.  \larcv will automatically enable this in, for example, read-only mode.  Alternatively, the user can have full, fine-grained control of which data products are written (and read) from disk through the configuration files.

\subsection{De-serialization, Pre-processing and Data Loading}

\larcv uses an on-demand deserialization procedure, where no data is loaded until it is requested from file.  A dedicated class, \texttt{IOManager}, keeps track of the current Event index as well as what objects are loaded from disk.  When a user requests a data product, with an associated producer label, \texttt{IOManager} will load the entire object for the current Event index from disk and store it in memory.  Subsequent calls return the in-memory objects without disk access until processing of the current Event is finished.

\larcv features a dynamic object storage system where users can create data product objects on-the-fly during runtime, as well as apply augmentation techniques to data products.  This is useful in particular for applying augmentation techniques for training deep neural networks, as the typical transforms (mirroring, rotations, etc) do not have easily accessible sparse algorithms available.

For training a neural network or similar batch-processing based workloads, a special interface to \larcv exists: the \texttt{QueueIO} class. This provides queued, in-the-background, data loading as well as an easy interface to the data products from \texttt{Python}.  In this mode, \larcv will start loading the next data from disk, and apply any desired processing, while the user's application is still processing the currently loaded data.  For GPU-based applications, this provides excellent overlap of CPU-based IO with GPU-based computing, and can reduce IO-related time to negligible levels.  The implementation relies on \texttt{std::future} objects in \texttt{C++}.  Additionally, the queued IO implementation is intrinsically scalable, as shown in Section~\ref{sec:performance}.  Users may dynamically configure the indexes of the next batch, or \larcv can randomize the dataset on the fly.  Sequential access is available as well, well suited for inference modes, and a hybrid ``random-block'' interface draws locally sequential, globally random sequences of events to improve disk access patterns without fully sacrificing dataset randomization.

\subsection{Visualization Tool}

Due to the unique data format and serialization scheme of \larcv dataproducts, we provide a python-based visualization tool, simply called \texttt{larcv-viewer} and available publically on GitHub \cite{larcv-viewer}. The viewer is also based on open-source tools, particularly \texttt{PyQtGraph} \cite{pyqtgraph} and \texttt{PyQt5} \cite{PyQt5}.  All images of data shown in this paper were produced with this visualization tool.