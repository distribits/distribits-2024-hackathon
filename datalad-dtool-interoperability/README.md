# Making datalad and dtool datasets interoperable

Ashwin Vazhappilly, Johannes L. Hörmann

## Short summary

Making datalad and dtool interoperable, i.e. 

1. create a dtool storage broker that operates on datalad datasets, and/or
2. create a datalad extension that dumps a dtool dataset
3. what to do about metadata?

## In detail

The story behind the dtool & dserver ecosystem is summed up in [story.rst](story.rst).

In short, it's a distributed data management ecosystem not unlike datalad, with a few prominent differences,

* datalad builds on git and git-annex, dtool at its core is a pure Python library, https://github.com/jic-dtool/dtoolcore
* dtool deliberately offers no versioning beyond a simple dataset freezing mechanism
* dtool datasets are defined by the way so-called storage brokers interact with infrastructure

The repository https://github.com/livMatS/dtool-demo illustrates some stereotypical dtool use case sessions on the command line.

A typical operation with dtool would be 

    dtool cp smb://some-windows-share/my-dataset s3://some-object-storage-bucket

to move datasets around. Datasets might be represented in different ways on the underlying infrastructure,
and how they are represented is defined by the storage brokers, e.g. https://github.com/jic-dtool/dtool-s3 or https://github.com/livMatS/dtool-smb,
with the constraint to have datasets in an as-plain-as-possible format, understandable and reconstructable even in the absence of any dtool-specific tools.

The [`DiskStorageBroker`](https://github.com/jic-dtool/dtoolcore/blob/36bab9447d8f9da014af0de65a7df65ff2342ae7/dtoolcore/storagebroker.py#L391-L772)
for handling datasets on the local file system illustrates this, it operates on a folder structure documented at https://peerj.com/articles/6562/#fig-1.

What stays the same across diverse infrastructure is the storage broker API for interacting with datasets,
defined in the [`BaseStorageBroker`](https://github.com/jic-dtool/dtoolcore/blob/36bab9447d8f9da014af0de65a7df65ff2342ae7/dtoolcore/storagebroker.py#L73C7-L227) class.

This makes storage brokers for representations in databases or via other middleware thinkable.

We would like to propose exploring the possibility of a datalad storage broker, i.e. the `DataLadStorageBroker`,
that implements the atomic operations on a dtool dataset by means of the [DataLad Python API](http://docs.datalad.org/en/stable/modref.html) where possible.
The vision is to make the command

    dtool cp file://path/to/my-dataset datalad://some-uri

result in a valid DataLad dataset with no information lost.

Alternatively, we could explore the possibility of a
[datalad extension](http://docs.datalad.org/en/stable/customization.html#writing-your-own-extensions)
that dumps a frozen dtool dataset as a snapshot of a versioned datalad dataset.

In both cases, we would have to think about the handling of metadata. 
