# PyNeuroTrace: Python code for Neural Timeseries

## Installation

This library can be installed directly from github, using pip:
```
pip3 install --upgrade "git+https://github.com/padster/pyNeuroTrace#egg=pyneurotrace&subdirectory=pyneurotrace"
```

## Reading Data

A number of options for loading data files are available within [files.py](https://github.com/padster/pyNeuroTrace/blob/master/pyneurotrace/pyneurotrace/files.py), including:
* `load2PData(path)` takes an experiment output file (e.g STEP_5_EXPT.TXT) and returns ID, XYZ location, and raw intensity values for each node in the experiment.
* `loadMetadata(path)` takes a metadata file (e.g. rscan_metadata_step_5.txt) and returns the stimulus start/stop samples, as well as the sample rate for the experiment.
* `loadTreeStructure(path)` takes a tree structure file (e.g. interp-neuron-.txt) and returns the mapping of node IDs to tree information about that node (e.g. node type, children, ...).

## Processing Data

Common per-trace processing filters are provided within [filters.py](https://github.com/padster/pyNeuroTrace/blob/master/pyneurotrace/pyneurotrace/filters.py), including:
* `deltaFOverF0(traces, hz, t0, t1, t2)` converts raw signal to the standard Delta-F-over-F0, using the technique given in Jia et al.
* `okada(traces)` reduces noise in traces by smoothing single peaks or valleys.
* `oasisSmooth(traces)` removes noise in traces by applying the [OASIS](https://github.com/j-friedrich/OASIS) algorithm and replace each trace with the best fit given by oasis. **(experimental)**

## Vizualization

A number of tools for vizualizing the data are provided within [viz.py](https://github.com/padster/pyNeuroTrace/blob/master/pyneurotrace/pyneurotrace/viz.py), including:
* `plotIntensity(data, hz, branches, stim, title)` generates a 2D intensity image, with rows being nodes and columns being samples of data. The intensity is normalized, ranging from black (lowest), to red, to yellow, and finally white (highest). Optionally, branch colors can be indicated on the left, and stimulus start/stop times can be shown below.
* `plotLine(data, hz, branches, stim, title, split)` generates line graphs for the provided traces. This accepts similar parameters to the intensity graph, as well as the ability to split lines into their own section of the y chart.
* `plotAveragePostStimIntensity(data, hz, stimOff, stimOn)` averages post-stimulus responses for off and on stimuli, and renders them for each node as an intensity graph.
* `plotAveragePostStimTransientParams` takes the response shape for all the nodes, fits a double-exponential curve, and shows histograms of the best parameters (A, t0, tA, tB) to get an idea of consistency across the nodes.
* `plotPlanarStructure` plots the node (X, Y) positions, and parent-child edges, plus branch colors.
* `planarAnimation` does similar to plotPlanarStructure, however the color of each node point is the intensity provided, and a movie is generated by iterating through all time points to give a visual representation of localized intensity changes.

## Event Detection (experimental)

Python implementations of the three algorithms discussed in Sakaki et al for finding events within Calcium events.
* `ewma(data, weight)` calculates the Exponentially-Weighted Moving Average for each trace, given how strongly to weight new points (vs. the previous average).
* `cusum(data, slack)` calculates the Cumulative Sum for each trace, taking a slack parameter which controls how far from the mean the signal the signal needs to be to not be considered noise.
* `matchedFilter(data, windowSize, A, tA, tB)` calculates the likelihood ratio for each sample to be the end of a window of expected transient shape, being a double exponential with amplitude A, rise-time tA, and decay-time tB.

The results of each of these three detection filters can then be passed through `thresholdEvents(data, threshold)`, to register detected events whenever the filter strength increases above the given threshold.
