---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.13.6
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

### Graph theory and machine learning for bioinformatics continued

  
* We used a Python library called [NetworkX](https://networkx.org/documentation/stable/tutorial.html)
* Examined different types of graphs, directed, undirected, weighted etc.
* Easy with sample data and few nodes, we will now learn to parse real datasets
* We will also warm up with Python so we can move to more advanced coding subsequently

---

### Example biological Graph data sources for bioinformatics research

* Biological General Repository for Interaction Datasets : [BioGrid](https://thebiogrid.org/) 
* European Bioinformatics Institute : [IntAct](https://www.ebi.ac.uk/intact/home) 
* Stanford Biomedical Network Dataset Collection : [SNAP](http://snap.stanford.edu/biodata/index.html) 

---

### Biological Graph data formats

A range of available formats from [BioGrid](https://wiki.thebiogrid.org/doku.php/downloads) 
The official specification of [PSI-MI TAB](https://psicquic.github.io/MITAB27Format.html) 

* Most intuitive is to use tab-delimited formats such as PSI-MI TAB
* We prefer the latest version of PSI-MI TAB (or at least >2.6)
* We can get the node annotations without additonal database referrals

---


### PSI-MI TAB in terminal 

![PSI-MI TAB 2.7 in terminal](https://i.ibb.co/WxywbSz/Screenshot-2022-02-07-7-47-42-PM.png) 


---

### PSI-MI TAB in Excel 


![PSI-MI TAB 2.7 in Excel](https://i.ibb.co/D4jHKts/Screenshot-2022-02-07-8-31-41-PM.png) 
[.tsv file 1 parse with Python in Collab](https://easyupload.io/xnrb66) 
[.tsv file 2 parse with Python in Collab](https://easyupload.io/nfycv0) 
[.tsv file 3 parse with Python in Collab](https://easyupload.io/rs5b9m) 

---

### Parsing PSI-MI TAB and creating a Graph with NetworkX 

* We will be using this [Collab Python notebook](https://colab.research.google.com/drive/1CQkYrIfRJFMRWmMR9MZ5W-yBj0QhxAMx?usp=sharing) 
* "File->Save Copy" the notebook under your Google account to edit or execute
* Remember the upload and copy path buttons to use .tsv from within Collab ![](https://i.ibb.co/d0nHpLG/colab-download.png) 
* And also see this [video](https://www.youtube.com/watch?v=6HFlwqK3oeo)

---

### Some more Google Collab tricks

* You can actually execute Unix commands, again see [Collab Python notebook](https://colab.research.google.com/drive/1CQkYrIfRJFMRWmMR9MZ5W-yBj0QhxAMx?usp=sharing) 
* Also refer to the [PSI-MI TAB fields](https://psicquic.github.io/MITAB27Format.html) 
* How many edges we have in total - can we tell before even using NetworkX ?

```python

!cut -f 1-2,17-18 /content/18467557.tsv | head -n 4

!cut -f 1-2,17-18 /content/18467557.tsv | sed -e 's/uniprotkb\://g' > /content/network.tsv

!wc -l network.tsv

```

---
### 

* Let's parse out the file and make a Graph with NetworkX
* We open the file for reading each line 'for line in f'
* The Python operations within the '[]' will generate a list with the results
* We split each line based on tabs ('\t') in a temporary array (the nested '[:2]')
* From that nested array we get the first two elements 0,1 ([:2] the 2 not inclusive) and store in 'pairset' array

```python

with open('/content/network.tsv', 'r') as f:

  pairset = [line.rstrip().split('\t')[:2] for line in f]

pairset[0:4]

```

---
### Draw the graphs

* Again do not forget the **'import network as nx'** etc. statements as needed in the beginning of your Collab code
* Remember the G.add_edges_from([(3, 4), (3, 5)]) to add edges and nodes simultaneously
* Why 'pairset[1:]' ? See also [Collab Python notebook](https://colab.research.google.com/drive/1CQkYrIfRJFMRWmMR9MZ5W-yBj0QhxAMx?usp=sharing) 


```python

G = nx.Graph()
G.add_edges_from(pairset[1:])
nx.draw(G, with_labels=True)
rkx.algorithms.tournament.hamiltonian_path¶


nodes = list(G.nodes)
print(nodes[0:10])

```

![](https://i.ibb.co/BtRHw5D/Screenshot-2022-02-08-3-07-57-PM.png)

---
### Example networks with the different .tsv files

![](https://i.ibb.co/YRfxRkL/download-1.png) 
![](https://i.ibb.co/NjBLx1m/download.png) 

---
### Connectivity of the Graph nodes

* Number of edges per node, a  histogram in descending order
* Note the special import of functions from networkx
* No information for which node has the highest number of connections

```python
from networkx.classes.function import info, degree_histogram

print(info(G))

deghist = degree_histogram(G)
deghist.sort(reverse=True)
print(deghist)

```

---
### Connectivity of the Graph nodes 

* This a normalized metric on how well connected a node is, [read definition here](https://towardsdatascience.com/graph-analytics-introduction-and-concepts-of-centrality-8f5543b55de3)
* Here we see which node the number the node corresponds to, using dictionary key-value pairs
* Notice the Python code for sorting the dictionary on its values

```python

from networkx.algorithms.centrality import degree_centrality

deg = degree_centrality(G)
print(deg)

sorted_deg = sorted( ( (value, key) for (key,value) in deg.items() ),key=None,reverse=True)
print(sorted_deg)

```

---
### Some more connectivity metrics 

* This returns a dictionary with the first node id as key, and dictionaries as values
* Each value dictionary, contains key-values with second node id and number of deletions for disconnect
* [See defintion and all algorithms here](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.connectivity.connectivity.all_pairs_node_connectivity.html) 

```python
from networkx.algorithms.approximation.connectivity import all_pairs_node_connectivity

allpaircon = all_pairs_node_connectivity(G)
print(allpaircon)

```

---
### Paths on the Graph

* Path from one node to other, and cyclic paths formed between nodes
* Note the 'casting' as list, because these methods return a 'generator'
* Saves memory and instantiated as list when requested as in code below

```python

from networkx.algorithms.simple_paths import all_simple_paths
from networkx.algorithms.cycles import cycle_basis

print(list(all_simple_paths(G, 'Q99836', 'Q9UGK3')))
print(list(cycle_basis(G)))

```

---
### Algorithmic / formal paths on the Graph

* [Hamiltonian path](https://en.wikipedia.org/wiki/Hamiltonian_path) visit each node exactly one time
* [Eulerian path](https://en.wikipedia.org/wiki/Eulerian_path) visit each edge only once (you can revisit nodes)

![height:12cm](https://i.ytimg.com/vi/CEOGcSCTar8/maxresdefault.jpg) 

[image credit and more information](https://www.youtube.com/watch?v=CEOGcSCTar8) 

---
### Algorithmic paths on networkx  

* Details in networkx documentation [here](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.tournament.hamiltonian_path.html)
* [and here](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.euler.is_eulerian.html)

```python
from networkx.algorithms.tournament import hamiltonian_path
from networkx.algorithms.euler import is_eulerian

print(is_eulerian(G))
hamiltonian_path(G.to_directed())

```

---
### Bonus: adjacency matrices and easy loading from files

* Print adjacency matrix of the graph
* Read graph automatically from a two-column file

```python

from networkx.linalg.graphmatrix import adjacency_matrix
adjacency_matrix(G).toarray()

from networkx.readwrite.adjlist import read_adjlist
!cut -f 1-2 /content/16365431.tsv > /content/network.tsv
G = read_adjlist('/content/network.tsv')
nx.draw(G, with_labels=True)

```

---
### The documentation of functions and algorithms of networkx

* [Functions](https://networkx.org/documentation/stable/reference/functions.html) 
* [Algorithms](https://networkx.org/documentation/stable/reference/algorithms/index.html) 


---
