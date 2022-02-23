---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.11.5
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

### Graph Theory and Network analysis in Bioinformatics

* In this lecture we will discuss basic Graph and Network Theory concepts and data structures
* Graphs or otherwise networks are one of the most common ways to represent bioinformatics data 
  * Interaction between genes and proteins (genetic regulation) can be represented as graphs
  * Protein structures can also be respensented as graphs denoting the chemical connections between atoms
  * Beyond molecular biology, interacting species for example can be represented as graphs in ecology

Some of the information we cover today can also be found in this [wiki article](https://en.wikipedia.org/wiki/Biological_network) 
and this [online course](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/network-analysis-in-biology/)

---

### What is a Graph

* A mathematical structure capturing binary relationships between objects called *nodes* or *vertices*
* The binary relationships are represented by links or *edges* between the *vertices*
* The *edges* can be of a specific type : *directed or undirected, weighted or not*
* Both *nodes* and *edges* can have attributes attached to them

---

### An example of a biological graph 
(credit [Hennah, Porteus](http://www.plosone.org/article/info%3Adoi%2F10.1371%2Fjournal.pone.0004906) CC BY 2.5 Licence)

![width:800px](https://upload.wikimedia.org/wikipedia/commons/7/72/Network_of_how_100_of_the_528_genes_identified_with_significant_differential_expression_relate_to_DISC1_and_its_core_interactors.png)

The different shapes of the nodes represent different attributes, the edges here are unweighted

---

### Protein-protein interaction (PPI) network graphs

* Protein-protein interaction networks (PINs) represent the physical relationship among proteins present in a cell
* Proteins are nodes, and their interactions are undirected edges
* Protein–protein interactions are essential to the cellular processes and well studied networks in biology
* Proteins with high degrees of connectedness (nodes with many links) are more likely to be essential for survival of the cell
* Source of this information and additional reading at this [wiki article](https://en.wikipedia.org/wiki/Biological_network)

---

### Let's create a graph in Python

*On Google Collab, run first "!pip install matplotlib"  and "!pip install networkx" (without the quotes)*

```python
import networkx as nx
import matplotlib.pyplot

G = nx.Graph()
G.add_node(1)
G.add_nodes_from([2, 3])
```

* We will use a Python library called [NetworkX](https://networkx.org/documentation/stable/tutorial.html)
* First we create an *undirected* graph, meaning it does not matter which direction we travel along the edges


---

### We only have nodes (vertices) 1,2,3 let's connect them with edges

```python
G.add_edge(1, 2, color='r',weight=2)

e = (2, 3) #another way to add an edge
G.add_edge(*e)  

G.add_nodes_from([4, 5]) #add two more vertices
G.add_edges_from([(3, 4), (3, 5)]) #add edges differently

nx.draw(G, with_labels=True, font_weight='bold') #labels used are node numbers 
```

---

### Let us add nodes with attributes

```python
G.add_node(6, molecule="protein") #now we also have vertex attribute
G.add_nodes_from([7,8,9], molecule="rna") #add a bunch of RNA vertices
G.nodes[1]["molecule"] = "protein"  #modify attributes of existing vertex
G.add_edges_from([(5,6),(5,7),(7,8),(8,9)])#connect these to the graph

labels = nx.get_node_attributes(G, 'molecule') #using custom labels for nodes
nx.draw(G, labels=labels, font_weight='bold') 

```
---

**Refer to these links for the graph coding examples and exercises**

Full documentation in NetworkX [tutorial](https://networkx.org/documentation/stable/tutorial.html)
and [NetworkX reference](https://networkx.org/documentation/stable/reference/classes/index.html)

---

### Graphs for complex bioinformatics data

![](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/wp-content/uploads/sites/64/2020/08/new-fig-3.png)
(credit [European Bioinformatics Institute](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-graph-types-and-edge-properties/) CC BY 2.5 Licence)

---

### Data structures for graphs data

![](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/wp-content/uploads/sites/64/2020/08/new-fig-4.png)
(credit [European Bioinformatics Institute](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-graph-types-and-edge-properties/) CC BY 2.5 Licence)

---

### Let's see this in Python code 

* Directed graphs represent directed control, for example biological regulation data
* For example Gene A, regulates Gene B
* Or Protein A controls gene expression of Gene C

```python

G1 = nx.DiGraph() #now we have a directed graph
G1.add_edges_from([(1,2), (2,3), (3, 4), (3, 5)]) #this adds nodes and edges 
G1.nodes[1]["name"] = "GeneA"
G1.nodes[2]["name"] = "GeneB"
G1.nodes[3]["name"] = "ProteinA"
G1.nodes[4]["name"] = "GeneC"
labels = nx.get_node_attributes(G1, 'name') #using custom labels for nodes nx.draw(G1, labels=labels, font_weight='bold') 

```
---

### 

![](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/wp-content/uploads/sites/64/2020/08/new-fig-5.png)<br>
(credit [European Bioinformatics Institute](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-graph-types-and-edge-properties/) CC BY 2.5 Licence)

* The degree is the number of edges that connect to a node
* The degree of each node is indicated and reflected in its size and colour
* Directed network nodes have two values : out-degree and in-degree 

---

### 

![](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/wp-content/uploads/sites/64/2020/08/new-fig-6.png)<br>
(credit [European Bioinformatics Institute](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-graph-types-and-edge-properties/) CC BY 2.5 Licence)

* Paths between nodes : how many nodes "hops" or steps
* Is used to model how information flows 
* Also to model how distant protein / genes control others 
* Algorithms to calculate shortest paths on the graph

---

### 

![](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/wp-content/uploads/sites/64/2020/08/new-fig-7.png)<br>
(credit [European Bioinformatics Institute](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-graph-types-and-edge-properties/) CC BY 2.5 Licence)

* Scale-free networks are those which have high connectivity hubs
* This means (sometimes equally) short paths from one section of the network to the other
* Additional concepts of centrality and transitivity, see [here](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-network-topology/) for more details

---

### 

```python

```
---


**Author: Konstantinos Krampis, 2021**<br><br> 
License: Creative Commons Attribution-NonCommercial-NoDerivs<br><br>
[CC BY-NC-ND](https://creativecommons.org/licenses/by-nc-nd/4.0/) ![](https://licensebuttons.net/l/by-nc-nd/3.0/88x31.png) 
