# Street Network Graph and Coverage Results

## 1. Loading and Understanding the Street Network Graph

The street network is stored as a serialized Python pickle file containing a NetworkX graph and a list of hydrant locations.

### Loading the Graph

```python
import pickle
import networkx as nx

with open("master_network.pickle", "rb") as f:
    G, Hyd_loc = pickle.load(f)
```

After loading:

- `G` is a NetworkX graph representing the street network.
- `Hyd_loc` contains the original hydrant locations used during graph construction.

### Graph Representation

The graph is a discretized representation of the street network. Each node corresponds to a location on the street network, and edges represent valid street connections between locations.

Node attributes contain both geographic coordinates and metadata. A typical node looks like:

```python
{
    'x': -118.38948026926336,
    'y': 34.05244384735747,
    'hasHydrant': False,
    'pos_original': (-118.38948026926336, 34.05244384735747),
    'pos': (970, 386)
}
```

### Node Attributes

| Attribute | Description |
|------------|-------------|
| `x` | Longitude of the node. |
| `y` | Latitude of the node. |
| `pos_original` | Original geographic coordinates `(longitude, latitude)`. |
| `pos` | Coordinates in a Mercator projection, used for visualization and spatial computations. |
| `hasHydrant` | Boolean indicating whether this node represents a hydrant location. |

### Hydrant Assignment

Hydrant locations were obtained from a separate dataset containing real hydrant latitude/longitude coordinates.

Because hydrants do not necessarily lie exactly on a graph node, a discretization step was applied during graph construction:

1. For each hydrant location, the nearest graph node was identified using geographic distance.
2. That node was selected as the hydrant representative.
3. The node's `hasHydrant` attribute was set to `True`.

This approximation is reasonable because the graph is sufficiently dense and node spacing is small relative to the scale of the analysis. As a result, the nearest-node assignment provides a close representation of the true hydrant location while simplifying graph-based computations.

### Accessing Node Information

Find all hydrant nodes:

```python
hydrant_nodes = [
    n for n, data in G.nodes(data=True)
    if data["hasHydrant"]
]
```

Inspect a node's attributes:

```python
node_id = 42
print(G.nodes[node_id])
```

Retrieve original latitude/longitude coordinates:

```python
lon, lat = G.nodes[node_id]["pos_original"]
```

Retrieve projected coordinates:

```python
x_proj, y_proj = G.nodes[node_id]["pos"]
```

---

## 2. Coverage Results

Coverage results for each tested sensor configuration are stored in files named:

```text
genesToCoverageXX.pickle
```

where `XX` denotes the corresponding SSPL value used for that experiment.

### Loading Coverage Data

```python
import pickle

with open("genesToCoverageXX.pickle", "rb") as f:
    genes_to_coverage = pickle.load(f)
```

After loading, `genes_to_coverage` is a dictionary mapping the number of deployed sensors to the resulting graph coverage.

### Dictionary Format

The dictionary has the following structure:

```python
{
    num_sensors: coverage_fraction,
    ...
}
```

where:

- `num_sensors` is the number of sensors deployed (out of the 48 available sensor locations).
- `coverage_fraction` is the fraction of nodes in `G` that are covered by that sensor configuration.

For example:

```python
{
    1: 0.124,
    5: 0.437,
    10: 0.681,
    20: 0.895,
    48: 1.0
}
```

would indicate that:

- Deploying 1 sensor covers 12.4% of graph nodes.
- Deploying 5 sensors covers 43.7% of graph nodes.
- Deploying 10 sensors covers 68.1% of graph nodes.
- Deploying 20 sensors covers 89.5% of graph nodes.
- Deploying all 48 sensors covers 100% of graph nodes.

Coverage values are computed relative to all nodes in the street network graph `G` described in Section 1.
