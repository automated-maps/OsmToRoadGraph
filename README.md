# OsmToRoadGraph v.0.4.3

[![Build Status](https://travis-ci.org/AndGem/OsmToRoadGraph.svg?branch=master)](https://travis-ci.org/AndGem/OsmToRoadGraph)
![Python application](https://github.com/AndGem/OsmToRoadGraph/workflows/Python%20application/badge.svg?branch=master)
[![codecov](https://codecov.io/gh/AndGem/OsmToRoadGraph/branch/master/graph/badge.svg)](https://codecov.io/gh/AndGem/OsmToRoadGraph)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

- [OsmToRoadGraph v.0.4.3](#osmtoroadgraph-v043)
- [Introduction](#introduction)
  - [Motivation](#motivation)
  - [Description](#description)
  - [Requirements](#requirements)
  - [Older Versions](#older-versions)
  - [Usage](#usage)
    - [Usage - Explanation](#usage---explanation)
    - [Example](#example)
    - [Output](#output)
      - [Output Format](#output-format)
        - [Example Road Network (*.pycgr)](#example-road-network-pycgr)
        - [Example Street Names (*.pycgr_names)](#example-street-names-pycgrnames)
    - [Configuring the Accepted OSM Highway Types](#configuring-the-accepted-osm-highway-types)
    - [Indoor Paths](#indoor-paths)
  - [Research](#research)

# Introduction

OSMtoRoadGraph aims to provide a simple tool to allow extraction of the road network of [OpenStreetMap](http://www.openstreetmap.org) files. It differentiates between three transportation networks: car, bicycle, and walking. The output data depends on the chosen parameters (which street highway types to consider, speed, ...).

## Motivation

OpenStreetMap provides free cartographic data to anyone. Data can be added and edited by anyone. However, using the road network contained in the OSM files is not straightforward. This tool aims to reduce the overhead of writing a parser for OSM files.

Below is an example of a visualization of the road network of the city of Bremen, Germany. The darker the shade of the street, the higher the maximum allowed speed.

<img src="https://raw.githubusercontent.com/AndGem/OsmToRoadGraph/master/examples/pycgr-to-png/bremen.png" width="350">

For details on how image was generated take a look into the [examples folder](https://github.com/AndGem/OsmToRoadGraph/tree/master/examples/pycgr-to-png). 


## Description

With this tool the data is being converted into easily parsable plaintext files that can be used by any application for further processing. For each transportation network type two output files are generated. One file contains the nodes (with coordinates), and the network edges, with length, direction and maximum speed (according to chosen network type). The second file contains street names that could be extracted for all edges contained in the first file.

## Requirements

- Python 3.6+/PyPy
- An OSM XML file

## Older Versions

Recently, breaking changes have been applied. If you require older versions please see the [releases](https://github.com/AndGem/OsmToRoadGraph/releases).

## Usage

```bash
Usage: run.py [options]

Options:
  -h, --help            show this help message and exit
  -f FILENAME, --file=FILENAME
  -n NETWORK_TYPE, --networkType=NETWORK_TYPE    (p)edestrian, (b)icycle, (c)ar, [default: pedestrian]
  -l, --nolcc
  -c, --contract
```

### Usage - Explanation

`-f` points to the input filename; the output files will be created in the same folder and using the name of the input file as prefix and adding information as suffix.

`-n` sets the network type. This influences the maximum speed saved for the edges. If you care only about connectivity set it to pedestrian.

`-l` if you set this option the graph will be output as a whole but _may_ contain unconnected components. By default the largest connected component is determined and the rest is dropped.

`-c` if you specify this option additional to the original graph, a second pair of filenames will be created containing the result of contracting all degree 2 nodes.

### Example

```bash
python run.py -f data/karlsruhe_small.osm -n p -v
```

### Output

The output will consist of two plaintext files. One file ending in `.pypgr`, `pybgr`, or `pycgr` depending on the network type selected; the other file will have the same ending with a `_names` as additional suffix. The first file contains the graph structure as well as additional information about the edge (length, max speed according to highway type, if it is a one-way street or not). The file ending with `_names` includes the street names for the edges.

#### Output Format

The structure of the road network output file is the following:

```
<HEADER LINES STARTING WITH A DASH(#) CHARACTER>
<number of nodes>
<number of edges>
<id> <lat> <lon>
...
<source_node_id> <target_node_id> <length> <street_type> <max_speed> <bidirectional>
...
```

The file begins with a header (some lines with a # character).

Then, two lines follow that contain the `number of nodes` and the `number of edges`.
After this, two larger blocks follow. In the first block, the nodes are being described, and in the latter the edges are described.
The first block consists of `<number of nodes>` many lines, and the second block consists of `<number of edges>` many lines.

Nodes of the graph are described by the following three parameters. Each nodes data is stored in one line, and the parameters are separated by a space:

- `<id>`: the node id (used later in the part where edges are to describe which nodes are connected by an edge)
- `<lat>`: latitude of the node
- `<lat>`: longitude of the node

Edges of the graph are described by 6 parameters. Each edge is stored in one line, and the parameters are separated by a space:

- `<source_node_id>`: the node id (see above) from which the edge originates
- `<target_node_id>`: the node id (see above) to which the edge leads to
- `<length>`: the length of the edge in meters (approximated)
- `<street_type>`: one of the OSM highway types (see: https://wiki.openstreetmap.org/wiki/Key:highway)
- `<max_speed>`: maximum allowed speed (if exists) in km/h [note: if no max speed is found a default value will be used]
- `<bidirectional>` indicates if an edge is bidirection. The value is `0` if it is a unidirectional road (from `source_node_id` to `target_node_id`), and otherwise it is `1`.

##### Example Road Network (*.pycgr)

```
# Road Graph File v.0.4
# number of nodes
# number of edges
# node_properties
# ...
# edge_properties
# ...
4108
4688
0 49.0163448 8.4019855
1 49.0157261 8.405539
2 49.0160334 8.4050578
...
531 519 93.87198088764158 service 10 0
524 528 71.98129087573543 service 10 1
528 532 22.134814663337743 service 10 1
532 530 12.012991347084839 service 10 1
530 531 12.76035560927566 service 10 1
531 529 14.981628728184265 service 10 1
529 501 77.18577344768484 service 10 1
501 502 10.882105497189313 service 10 1
75 405 14.312976598760008 residential 30 1
405 206 44.642284586584886 residential 30 1
...
```

##### Example Street Names (*.pycgr_names)

```
Hölderlinstraße
Hölderlinstraße
Hölderlinstraße

Kronenstraße
Kronenstraße
Kronenstraße

Zähringerstraße
Zähringerstraße
```

Each line consists of a street name. The number in which a line is corresponds to the edges index. In this example this means, that Hölderlinstraße is the street name of edges 0, 1, 2. The absence of a name in line 4 indicates that edge 3 has no street name. Edges 4, 5, 6 have street name Kronenstraße, and so on...

### Configuring the Accepted OSM Highway Types

The application comes with a set of standard configuration to parse `only` some OSM ways that have the tag `highway=x` where `x` is a [highway type](https://wiki.openstreetmap.org/wiki/Key:highway) [notable excepting is the `pedestrian_indoors`, see below for an explanation].
You can change the behavior of this program by changing the values (removing unwanted, and adding missing values) in the `configuration.py`.
In this file you can also modify the speed limit that will be written to the output file.

### Indoor Paths

By default elements tagged by the [Simple Indoor Tagging](https://wiki.openstreetmap.org/wiki/Simple_Indoor_Tagging) approach are being ignored.
To enable to also to extract these paths replace in `configuration.py` the line

```python
    accepted_highways['pedestrian'] =  set(["primary", "secondary", "tertiary", "unclassified", "residential", "service", "primary_link", "secondary_link", "tertiary_link", "living_street", "pedestrian", "track", "road", "footway", "steps", "path"])
```

with

```python
    accepted_highways['pedestrian'] =  set(["primary", "secondary", "tertiary", "unclassified", "residential", "service", "primary_link", "secondary_link", "tertiary_link", "living_street", "pedestrian", "track", "road", "footway", "steps", "path", "pedestrian_indoor"])
```

Note that the change is only the addition of `pedestrian_indoor` in this list.

## Research

[Temporal Map Labeling: A New Unified Framework with Experiments](http://i11www.iti.uni-karlsruhe.de/temporallabeling/)
