---
layout: post
title: Working with Siemens physio (.puls) files
author: Michał Szczepanik
tags: ['python', 'neuroscience', 'tips & tricks']
date: 2020-12-17 16:25
---

Siemens Physiological Monitoring Unit uses its own format for data storage. Conveniently, it produces text files with a fairly simple representation, which should be easy to parse. I only worked with pulse oximetry, but similar rules should hold for other physio data. This is a compilation of external sources, own experiences and code snippets.

## Toolboxes

Before we dig into the files: you may find what you already need in one of the following toolboxes (with which I have no experience), both in Matlab:
* [Physiological Log Extraction for Modelling](https://sites.google.com/site/phlemtoolbox/) (PhLEM) - a toolbox to generate physiological covariates to be included as regressors in SPM GLM
* [PhysIO toolbox](https://gitlab.ethz.ch/physio/physio-doc), part of Translational Algorithms for Psychiatry-Advancing Science (TAPAS) collection

## Sources

The information is out there, but it was not very easy to find and not always consistent (no doubt due to different variations of the format produced by different scanners or software versions). My primary references are the following:

1. [HCP wiki](https://wiki.humanconnectome.org/display/PublicData/Understanding+Timing+Information+in+HCP+Physiological+Monitoring+Files)
2. [PART toolbox website](https://www.mccauslandcenter.sc.edu/crnl/tools/part) from McCausland center (by Chris Rorden)
3. [Documentation for PhysIO toolbox](https://gitlab.ethz.ch/physio/physio-doc/-/wikis/MANUAL_PART_READIN) from ETH Zurich
4. [GKAguirre lab wiki](https://cfn.upenn.edu/aguirre/wiki/public:pulse-oximetry_during_fmri_scanning)

## File structure

Based on what I've seen, some details may depend on hardware & software versions used, but the general principles should be the same. The example below is based on the data obtained on the Trio scanner at the Laboratory of Brain Imaging. At the end I will also point out differences in the data collected on a Prisma in a different lab.

The .puls file is just a text file, in which the first line contains all the data points (!), and the following lines contain metadata. It looks like this:

```
1 2 40 280 1102 1069 1031 1017 1136 1449 …
ECG  Freq Per: 0 0
PULS Freq Per: 100 595
RESP Freq Per: 0 0
EXT  Freq Per: 0 0
ECG  Min Max Avg StdDiff: 0 0 0 0
PULS Min Max Avg StdDiff: 448 1298 607 2
RESP Min Max Avg StdDiff: 0 0 0 0
EXT  Min Max Avg StdDiff: 0 0 0 0
NrTrig NrMP NrArr AcqWin: 0 0 0 0
LogStartMDHTime:  54133082
LogStopMDHTime:   55183837
LogStartMPCUTime: 54132545
LogStopMPCUTime:  55179315
6003
```

The first four (?) digits in the first line are probably acquisition parameters (the meaning of which I don't know), the remaining are signal values interspersed with some markers (values < 5000 are data samples, values >= 5000 are markers that have to be sifted out). Values 5003 and 6003 are apparently used to mark the beginning and end of the metadata (5003 ends the first line). Sampling frequency for our setup is 50 Hz (but it can be easily verified).

There are plenty 5000 markers in the signal and I am not sure what they represent - judging by their location just after signal peaks, they seem to be heart beat markers[^2] inserted by the software (presumably analogous to the red triangles shown on the technician's console while monitoring the signal). Even if that is the case, you will most likely want to use a peak detection method from your favourite data analysis suite instead.

In the metadata, the crucial entries to watch for are the MDH Times[^1], expressed in milliseconds after midnight. Although notation is different, they are taken from the same clock as the time values in DICOM files and can be used for synchronising the two modalities.

## Loading the data using python

Numpy and pydicom libraries are essential; we will also use the built-in itertools:

~~~ python
import numpy as np
import pydicom
from itertools import islice
~~~

Let's start by defining paths to the pulse file and the first dicom file of a given series:

~~~ python
pulse_file_name = path_to_the_pulse_file
dicom_file_name = path_to_the_dicom_file
~~~

Loading the signal and dropping the markers is trivial, just remember to specify `dtype` & `max_rows`:

~~~ python
data = np.loadtxt(pulse_file_name, dtype=np.int, max_rows=1)
data = data[4:]  # drop parameters stored at the beginning
data = data[np.where(data < 5000)]  # drop markers
~~~

We can read all the metadata (although we only need `LogStartMDHTime`) into a dictionary:

~~~ python
meta = {}
with open(pulse_file_name) as f:
    for line in islice(f, 1, None):  # read from second line to the end
        try:
            k, v = line.split(':')
            meta[k.strip()] = v.strip()
        except ValueError:
            # don’t care about entries which can't be split into key: value
            pass
~~~

The time needs to be converted from string to a number, we can also convert it from miliseconds to seconds:

~~~ python
pulse_time_sec = int(meta['LogStartMDHTime']) * 0.001
~~~

Now, to read the time from dicom header we can use pydicom. I am using field (0008, 0032) Acquisition Time:

~~~ python
ds = pydicom.dcmread(dicom_file_name)
dicom_time = ds.AcquisitionTime
~~~

The time looks like this: `'090029.930000'`. The first two digits are hours, next two are minutes, next two - seconds. Four digits after the dot are _tics_ (1/10000 of a second), and the two trailing zeros can be ignored. A conversion function (translating into seconds) can look as follows:

~~~ python
def dicomtime2sec(t):
    h = int(t[:2])
    m = int(t[2:4])
    s = int(t[4:6])
    tics = int(t[7:11])

    return(h * 3600 + m * 60 + s + tics * 0.0001)
~~~

We will use this function to convert the value into seconds after midnight (same as physio recordings) and then calculate the offset between the recordings. Take care with studies lasting through midnight!

~~~ python
dicom_time_sec = dicomtime2sec(dicom_time)
delta_time = pulse_time_sec - dicom_time_sec
~~~

With that, we have all we need (pulse oximetry signal and its start time relative to the MRI data)!

Finally, we can verify the sampling frequency by dividing the number of samples by the duration read from metadata (likely it won't be _exactly_ 50 Hz, but it should come negligibly close):

~~~ python
duration = ( int(footer['LogStopMDHTime']) - int(footer['LogStartMDHTime']) ) / 1000 # in seconds
calculated_fs = data.shape[0] / duration
~~~


The result can now be saved in a desired format for further processing. When exporting, consider using the [BIDS convention for physiological recordings](https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/06-physiological-and-other-continuous-recordings.html), which uses a gzip compressed tsv file for data (tip: use `numpy.savetxt` - for filenames ending in `.gz`, the file is automatically compressed) and a json file to store sampling frequency and relative start time (tip: use `json.dump` from built-in json library).

This is left as an exercise for the reader, and so is dealing with cases when you have to divide one pulse recording made across several MRI runs.

## Prisma - note on the differences

I also had a chance to take a look at data from another lab, which has a Prisma scanner. It was _same, but different_. Most notably, it had text metadata inserted _inbetween_ the data values. Luckily, the marker convention seems to be clear, with 5002 and 6002 values used to mark the beginning and end of the inserted information. Interestingly, the sampling frequency turned out to be 400 Hz in this case. Beginning of the data may look as follows:

~~~
1 2 40 280 5002 LOGVERSION_PULS   1 6002 3185 3167 < ... about 200 datapoints ...> 5002 uiHwRevisionPeru/ucHWRevLevel: 0, uiPartNbrPeruPub: 0, uiHwRevisionPpu/ucSWSubRevLevel: 0, uiPartNbrPpuPub: 0, uiSwVersionPdau/ucSWMainRevLevel: 0 6002 1395 1391 ... 
~~~

The `numpy.loadtxt` function used previously will obviously crash on such input, luckily the most basic way of processing won't be much slower. Let's read the line, split by spaces and collect one element at a time (pausing collection for items between 5002 and 6002):

~~~ python
def read_pulse(filename):
    with open(filename) as f:
        line = f.readline()
        
    elements = line.split()
    
    read = True
    data = []
    for elem in elements:
        if elem == '5002':
            read = False
        elif elem == '6002':
            read = True
        elif read == True:
            data.append(int(elem))
    return np.array(data)
~~~

---

[^1] According to the website for PART, MDH is the DICOM clock, while MPCU is the physio clock. Of the sources, only the Aguirre wiki suggests using MPCU, which is likely a mistake.

[^2] Aguirre wiki suggests that they are scanner pulses marking TR, but the timing doesn't check out, so maybe it depends on software version.
