# proteolizard-algorithm
### A collection of algorithms and tooling to process ion-mobility mass-spectrometry raw-data

<p align="center">
  <img src="logo.png" alt="logo" width="500"/>
</p>

This repository is part of the `proteolizard` project, a free and open-source solution 
for raw-data access, algorithms and raw-data visualization of mass spectrometry data generated with 
the bruker timsTOF device.

We are a relatively small team of developers and therefore decided to keep things loosely coupled. 
This means that 

* **data access**  : [`proteolizard-data`](https://github.com/theGreatHerrLebert/proteolizard-data)
* **algorithms**   : [`proteolizard-algorithm`](https://github.com/theGreatHerrLebert/proteolizard-algorithm) 
* **visualization**: [`proteolizard-vis`](https://github.com/theGreatHerrLebert/proteolizard-vis) 

are made available at different repositories. 
This makes it easier for us to develop all pieces independently.
We try to keep dependencies as small as possible, which should allow you to exchange parts of your 
custom pipelines against other data-access backends such as 
[`timspy`](https://github.com/MatteoLacki/timspy) or 
[`alphatims`](https://github.com/MannLabs/alphatims).

Development is still ongoing. 
If you experience weird behaviour, bugs or errors please let us know!


## Why proteolizard-algorithm?
`proteolizard-algorithm` provides you with algorithms and tools that are tailored to deal with the huge 
amount of raw-data generated by liquid chromatography coupled to ion-mobility tandem mass-spectrometry (LC-IMS-MS-MS).
The additional recording of ion-mobility adds another dimension to experiments while 
data-sparsity increases as well. 
This makes a lot of traditional approaches used for LC-MS-MS processing either 
too slow or their design unsuited for these datasets. 

Our goal is to translate ideas developed in other disciplines in data science that have to deal with related problems.
We especially want to make use of modern hardware such as multicore systems and GPU parallelization. 

## Navigation
* [**Build and install proteolizard-algorithm**](#build-and-install-proteolizard-algorithm)
* [**Locality Sensitive Hashing (LSH)**](#locality-sensitive-hashing-(lsh))
* [**Clustering**](#clustering)
* [**Supervised (Deep) Learning**](#supervised-(deep)-learning)

---
### Build and install proteolizard-algorithm
We highly recommend to install all libraries that are part of the `proteolizard` project into a python [virtual environment](https://docs.python.org/3/tutorial/venv.html) or [conda environment](https://docs.conda.io/en/latest/).

To use `proteolizatd-algorithm`, you will need to install [`proteolizard-data`](https://github.com/theGreatHerrLebert/proteolizard-data) first. After that, build the C++ shared library for python:

```sh
shell> git clone https://github.com/theGreatHerrLebert/proteolizard-algorithm
shell> cd proteolizard-algorithm
```

```sh
shell> mkdir build && cd build
shell> cmake ../cpp -DCMAKE_BUILD_TYPE=Release
shell> make
```

Or, if you did not install `proteolizard-data` into a global install directory, you also need to set CMAKE_PREFIX_PATH to the same installation prefix used for proteolizard-data:

```sh
shell> mkdir build && cd build
shell> cmake ../cpp -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=path/to/proteolizard-data/install
shell> make
```

```sh
shell> cmake --install . --prefix=some/prefix/path
```
---
### Locality Sensitive Hashing (LSH)
LSH is a stochastic technique to find similar objects, where similarity is estimated using a family of hash functions 
that are tailored to approximate some similarity measure. One of its main advantages over other algorithms is the fact
that similar pairs can be found in linear time, trading a guarantee to find all similar objects against high
probability of detection.

`proteolizard-algorithm` implements approximation of cosine similarity of mass spectra. To do so, it allows you to
generate a set of keys for mz spectra in a vectorized representation. This keys can then be used for detection of 
self-collision, reference search or generally anything related to distance matrices like clustering. Key calculation
is based on [`tensorflow`](https://www.tensorflow.org/) Tensors and can therefore be put onto the GPU if you have a 
[CUDA](https://developer.nvidia.com/cuda-toolkit) enabled NVIDIA card and 
[cuDNN](https://developer.nvidia.com/cudnn) is available in your environment. 

We will briefly go over how LSH is performed for timsTOF data.

**TODO**: explain and show workflow plot.

If you want to learn more about LSH in context of mass spectrometry, have a look at 
Bob et al.[^fn1] or 
Wang et al.[^fn2][^fn3]

```python
import numpy as np
import tensorflow as tf

from proteolizarddata.data import PyTimsDataHandle, TimsFrame, MzSpectrum
from proteolizardalgo.hashing import TimsHasher, IsotopeReferenceSearch, ReferencePattern
from proteolizardalgo.utility import create_reference_dict, get_refspec_list, get_ref_pattern_as_spectra

# create a data handle and read a precursor frame
dh = PyTimsDataHandle('/path/to/data.d')
frame = dh.get_frame(dh.precursor_frames[250])

# create a set of dense windows indexed by scan and mz-bin
scan, mz_bin, W = frame.get_dense_windows(window_length=4, resolution=2, min_peaks=5, 
                                          min_intensity=50, overlapping=True)

# create a spectrum hasher
# by picking a fixed seed, you can guarantee that keys can be reproduced
hasher = TimsHasher(trials=256, len_trial=22, seed=42, num_dalton=4, resolution=2)

# calculate trials number of keys, each having len_tral bits for each window
K = hasher.calculate_keys(W)

print(K)
```
This will give you:
```python
<tf.Tensor: shape=(10682, 512), dtype=int32, numpy=
array([[ 362167, 3700797, 3061941, ..., 1147456, 1968934,   98534],
       [2538463, 3497250, 2595794, ..., 2643667, 2048648, 3815282],
       [2003423, 3821990, 2528830, ..., 1697390, 1763353, 1735530],
       ...,
       [2898374, 1166177, 1438584, ..., 2115578,  769518,  448939],
       [1382299, 3202454, 3824606, ..., 2843920, 1615614, 3689973],
       [ 877019, 3258715, 4001803, ..., 1603336, 2742681, 2790119]],
      dtype=int32)>
```
where shape = (number_windows, number_keys_per_window).

---
### Clustering
DUMMY

---
### Supervised (Deep) Learning
Zohora et al.[^fn4][^fn5]

---
[^fn1]: Locality-sensitive hashing enables efficient and scalable signal classification in high-throughput mass spectrometry raw data.
bioRxiv 2021. https://doi.org/10.1101/2021.07.01.450702

[^fn2]: A Fast and Memory-Efficient Spectral Library Search Algorithm Using Locality-Sensitive Hashing. 
Proteomics, 2020. https://doi.org/10.1002/pmic.202000002

[^fn3]: msCRUSH: Fast Tandem Mass Spectral Clustering Using Locality Sensitive Hashing.
journal of proteome, 2019. https://pubs.acs.org/doi/10.1021/acs.jproteome.8b00448

[^fn4]: DeepIso: A Deep Learning Model for Peptide Feature Detection from LC-MS map.
Nature scientific reports, 2019. https://doi.org/10.1038/s41598-019-52954-4

[^fn5]: Deep neural network for detecting arbitrary precision peptide features through attention based segmentation.
Nature scientific reports, 2021. https://doi.org/10.1038/s41598-021-97669-7
