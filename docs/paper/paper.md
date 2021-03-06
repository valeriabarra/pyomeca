---
title: "`pyomeca`: An Open-Source Framework for Biomechanical Analysis"
tags:
  - python
  - biomechanics
  - electromyography
  - kinematics
  - dynamics
authors:
  - name: Romain Martinez
    orcid: 0000-0001-9681-9448
    affiliation: "1"
  - name: Benjamin Michaud
    orcid: 0000-0002-5031-1048
    affiliation: "1"
  - name: Mickael Begon
    orcid: 0000-0002-4107-9160
    affiliation: "1"
affiliations:
  - name: School of Kinesiology and Exercise Science, Faculty of Medicine, University of Montreal, Canada
    index: 1
date: 2 June 2020
bibliography: paper.bib
---

# Statement of Need

Biomechanics is defined as the study of the structure and function of biological systems by means of the methods of mechanics [@Hatze1974-zc].
While musculoskeletal biomechanics branches into several subfields, the data used are remarkably similar.
The processing, analysis and visualization of these data could therefore be unified in a software package.
Most biomechanical data characterizing human and animal movement appear as temporal waveforms representing specific measures such as muscle activity or joint angles.
These data are typically multidimensional arrays structured around labels with arbitrary metadata (\autoref{fig:biomech-data}).
Existing software solutions share some limitations.
Some of them are proprietary [@Damsgaard2006-gq] or based on closed-source programming language [@Dixon2017-co; @Muller2019-cx].
Others do not leverage labels and metadata [@Walt2011-em; @Hachaj2019-tk; @Virtanen2020-zv].
`pyomeca` is a Python package designed to address these limitations.
It provides basic operations useful in the daily workflow of a biomechanical researcher such as reading, writing, filtering and plotting, but also more advanced biomechanical routines geared towards rigid body mechanics and signal processing.
By offering a single, efficient and flexible implementation, `pyomeca` standardizes these procedures, freeing up valuable research time, thereby allowing researchers to focus on the scientific research questions at hand.

![An example of musculoskeletal biomechanical data with skin marker positions.
These data are inherently multidimensional and structured around labels.
Metadata are also needed to inform about important features of the experiment.\label{fig:biomech-data}](fig/biomech-data.pdf)

# Summary

As a Python library, `pyomeca` enables extraction, processing and visualization of biomechanical data for use in research and education.
It is motivated by the need for simpler tools and more reproducible workflows allowing practitioners to focus on their specific interests and leaving `pyomeca` to handle the computational details for them.
`pyomeca` builds on the core scientific Python packages, in particular `numpy` [@Walt2011-em], `scipy` [@Virtanen2020-zv], `matplotlib` [@Hunter2007-fv] and `xarray` [@Hoyer2017-sf].
By providing labeled querying and computation, efficient algorithms and persistent metadata, the integration of `xarray` facilitates usability, which seems to be relevant in a context where scientists do not always receive extensive training in programming.
`xarray` is designed as a general-purpose library and tries to avoid including domain specific functionalities --- but inevitably, the need for more domain specific logic arises.
`pyomeca` provides a biomechanics layer that supports specialized file formats (`c3d`, `mat`, `trc`, `sto`, `mot`, `csv` and `xlsx`) and implements signal processing and matrix manipulation routines commonly used in musculoskeletal biomechanics.
We believe `pyomeca` is a solid foundation for more advanced research implementations and could, in the future, provide functions for processing and extracting meaningful clinical outputs from the raw data.
`pyomeca` was written with the Unix philosophy in mind, that is to design each program to do one thing well and expect the output of every program to become the input to another, as yet unknown, program.
As such, it is written in a modular, object-oriented way --- which makes it extensible and easy to develop --- and it interacts well with other projects under the [`pyomeca umbrella`](https://github.com/pyomeca), namely [`ezc3d`](https://github.com/pyomeca/ezc3d), [`pyosim`](https://github.com/pyomeca/pyosim) or [`biorbd-viz`](https://github.com/pyomeca/biorbd-viz).
`pyomeca` follows software best practices by being fully tested, linted and type annotated --- ensuring that the package is easily distributable and modifiable.
In addition to the [static documentation and API reference](https://pyomeca.github.io/), `pyomeca` includes a set of Jupyter Notebooks with examples.
These notebooks can be read and executed by anyone with only a web browser through [binder](https://mybinder.org/).

# Features

`pyomeca` inherits from the `xarray` feature set, which includes label-based indexing, arithmetic, aggregation and alignment, resampling and rolling window operations, plotting, missing data handling and out-of-core computation.
In addition, `pyomeca` has four data structures built upon `xarray`.
Each structure is associated with a specific biomechanical data type:

- `Angles`: joint angles,
- `Rototrans`: rototranslation matrix,
- `Analogs`: generic signals such as electromyographic (EMG) and force signals or any other analog signals,
- `Markers`: skin markers positions.

While there are technically dozens of functions implemented in `pyomeca`, one can generally group them into two distinct categories: object creation and data processing.

## Object Creation

The starting point for working with `pyomeca` is to create an object with one of the specific methods associated with the different classes available.
`pyomeca` offers several ways to create these objects: by directly specifying the data, by sampling random data from distributions, by converting other data structures or by reading files (\autoref{fig:object-creation}).

![`pyomeca` offers several ways to create specialized data structures: from scratch (orange), from random data (red), from other data structures (blue) or from files (green).\label{fig:object-creation}](fig/object-creation.pdf)

## Data Processing

`pyomeca`'s main functionality is to offer dedicated biomechanical routines.
These features can be broadly grouped into different categories: filtering, signal processing, normalization, matrix manipulation and file output functions (\autoref{fig:data-processing}).

![`pyomeca` data processing capabilities are available through the `meca` `DataArrayAccessor` (e.g. `array.meca`) that allow implementing domain specific methods on `xarray` data objects.
These methods can be categorized into filters (orange), signal processing (red), normalization (blue), matrix manipulation (green) and file output (purple) routines.\label{fig:data-processing}](fig/data-processing.pdf)

## A Biomechanical Example: Electromyographic Pipeline

`pyomeca` has documented examples for different biomechanical tasks such as getting Euler angles from a rototranslation matrix, creating a system of axes from skin markers positions or setting a rotation or a translation.
Another typical task concerns EMG data processing.
Using `pyomeca`, one can easily extract (\autoref{fig:ex-1-raw}), process (\autoref{fig:ex-2-processed}) and visualize (\autoref{fig:ex-3-aggr}, \autoref{fig:ex-4-box} and \autoref{fig:ex-5-corr}) such data.

```python
from pyomeca import Analogs

emg = Analogs.from_c3d("data.c3d")
emg.plot(x="time", hue="channel")
```

![Biomechanical data are often stored in the `c3d` binary file format.
Thanks to the `ezc3d` library [@ezc3d], `pyomeca` can easily read these files and visualize them with the `matplotlib` interface provided by `xarray`.
\label{fig:ex-1-raw}](fig/ex-1-raw.pdf)

```python
emg_processed = (
    emg.meca.band_pass(order=2, cutoff=[10, 425])
    .meca.center()
    .meca.abs()
    .meca.low_pass(order=4, cutoff=5)
    .meca.normalize()
)
emg_processed.plot(x="time", col="channel", col_wrap=3)
```

![EMG data analysis consists of a series of signal processing steps that can be carried out by `pyomeca` in a clear and modular way.\label{fig:ex-2-processed}](fig/ex-2-processed.pdf)

```python
import matplotlib.pyplot as plt

_, axes = plt.subplots(ncols=2)

emg_processed.mean("channel").plot(ax=axes[0])
emg_processed.plot.hist(ax=axes[1], bins=50)
```

![It is straightforward to represent the average profile of the EMG signal (left) or the distribution of EMG activations (right) thanks to `xarray`.\label{fig:ex-3-aggr}](fig/ex-3-aggr.pdf)

```python
emg_dataframe = emg_processed.meca.to_wide_dataframe()
emg_dataframe.plot.box(showfliers=False)
```

![`pyomeca` offers a method to convert the data structure into a `pandas` dataframe [@McKinney2010-lm].
This allows users to further extend the plot possibilities using the visualization built into `pandas` itself, such as boxplot.\label{fig:ex-4-box}](fig/ex-4-box.pdf)

```python
emg_dataframe.corr().style.background_gradient().set_precision(2)
```

![By using a `pandas` dataframe, users also benefit from its broad range of IO tools and statistical methods, such as computing the correlation matrix between the different muscles.\label{fig:ex-5-corr}](fig/ex-5-corr.pdf)

# Research Projects Using `pyomeca`

You can find an [up-to-date list of research projects using `pyomeca`](https://pyomeca.github.io/about/#papers-citing-pyomeca) on the static documentation.

# Acknowledgements

`pyomeca` is an open-source project created and supported by the Simulation and Movement Modeling (S2M) lab located in Montreal.
We thank the contributors that helped build `pyomeca`.
You can find an [up-to-date list of contributors](https://github.com/pyomeca/pyomeca/graphs/contributors) on GitHub.
We also would like to extend thanks to the contributors of the libraries used to build `pyomeca` --- particularly `numpy` [@Walt2011-em], `scipy` [@Virtanen2020-zv], `matplotlib` [@Hunter2007-fv] and `xarray` [@Hoyer2017-sf].

# References
