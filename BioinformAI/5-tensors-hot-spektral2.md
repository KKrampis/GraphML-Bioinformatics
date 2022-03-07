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

### One-hot encoding of protein annotations

![](https://i.ibb.co/ZSs4NGz/Screenshot-2022-02-19-7-38-54-PM.png)

* Annotations with 5 words, the dictionary provides index for matching the 1's with the set of words
* Tensor: 3-dimensional matrix, (annotation length) x (size of word set) x (number of annotations) 

---
### And now onwards to first building the index for the words

```python

import numpy as np
import gensim
from gensim import corpora

annotations = ['This protein phosporylates a gene','This protein catalyzes a reaction', \
'This gene controls RNA expression', 'This gene controls a gene']

annotwords = [[word for word in annotation.split()] for annotation in annotations]
annotdict = corpora.Dictionary(annotwords)
annotensor = np.zeros(shape = (len(annotations), len(annotations[0].split(' ')), max(annotdict.token2id.values()) + 1))  

for i, annotation in enumerate(annotations): 
  for j, word in list(enumerate(annotation.split())):
    
    index = annotdict.token2id.get(word)
    annotensor[i, j, index] = 1.   

print(annotwords,"\n");print(annotdict.token2id,"\n");print(annotensor)
```

---

### Graph Convolutional Networks

* Graph Convolutional Networks - GCNs, a type of Artificial Neural Networks 
* We will utilize vectorized, one-hot encoded annotation data on each node on our graphs
* Matrix multiplication is row to column, (k,l) x (l,m) = (k,m) inner dimensions must match
![width:600](https://miro.medium.com/max/1400/1*YGcMQSr0ge_DGn96WnEkZw.png)

---

### Tensor operations with numpy.array

```python
sliceanot = np.sum(annotensor,axis=1)
print(sliceanot,"\n")
print(sliceanot[0:4, 0:5],"\n")

anot = np.sum(sliceanot[0:4, 0:5],axis=1)
anotvec = np.reshape(anot,[4,1])

print(anotvec,"\n")
```
![](https://i.ibb.co/ZSs4NGz/Screenshot-2022-02-19-7-38-54-PM.png)

---

### A graph with the compressed annotations as node attributes

```python
import networkx as nx

A = np.matrix([
    [0, 1, 0, 0, 0],
    [0, 0, 0, 1, 0], 
    [0, 1, 0, 0, 1],
    [1, 0, 1, 0, 0],
    [1, 1, 0, 0, 0]],
    dtype=float)

G = nx.convert_matrix.from_numpy_matrix(A,create_using=nx.DiGraph)

# I have added on extra node that is not annotated, preventing index error
anotvec = np.append(anotvec, 0)
anotvec = np.reshape(anotvec,[5,1])

for i in G.nodes:
  G.nodes[i]["annotval"] = (int(anotvec[i]),'ID=' + str(i))
    
labels = nx.get_node_attributes(G, 'annotval') 
nx.draw_shell(G, arrows=True, labels=labels)
```
---

### Update node attributes based on neighboors


```python
X = anotvec
print(A,"\n")
print(X,"\n")

anotprop = A * X
print(anotprop,"\n")

for i in G.nodes:
  G.nodes[i]["annotval"] = (int(anotprop[i]),'ID=' + str(i))

labels = nx.get_node_attributes(G, 'annotval') 
nx.draw_shell(G, arrows=True, labels=labels)
```
* Remember the numbers represent "codified" annotations (although very simplified in our example)
* Rename vector with the annotation numbers "X" to match tutorial we follow [here](https://towardsdatascience.com/how-to-do-deep-learning-on-graphs-with-graph-convolutional-networks-7d2250723780) and [here](https://towardsdatascience.com/understanding-graph-convolutional-networks-for-node-classification-a2bfdb7aba7b)  
---

### Adding self-loops

```python
I = np.matrix(np.eye(A.shape[0]))
print(I,"\n")

A1 = A + I
print(A1,"\n")

anotvecself = A1 * X

for i in G.nodes:
	G.nodes[i]["annotval"] = (int(anotvecself[i]),'ID=' + str(i))

labels = nx.get_node_attributes(G, 'annotval') 
nx.draw_shell(G, labels=labels)
```

* Add the identity matrix, we add self-loops in the graph, sum up also the annotation value of each node

---

### Normalizing for the number of edges per node

```python

D = np.array(np.sum(A, axis=0))[0]
D = np.matrix(np.diag(D))
anotvecscaled = D**-1 * A1 * X

print(D,"\n");print(A,"\n")
print("D(5x5) * A1(5x5) * X(5x1) = annotvecscaled(5x1)\n")
print(D**-1,"\n");print(A1,"\n");print(X,"\n")
print(anotvecscaled,"\n")

for i in G.nodes:
	G.nodes[i]["annotval"] = (int(anotvecscaled[i]),'ID=' + str(i))

labels = nx.get_node_attributes(G, 'annotval') 
nx.draw_shell(G, labels=labels)
```

* D sums the number of edges per node (the 1's in each column of the adjacency matrix A)
* Inverse number of edges on diag(D), scales / normalizes annotation number assigned to each node during the matrix multiplication

---
### A step back to consider our algorithmic approach

![](https://i.ibb.co/FKRHNBL/Screenshot-2022-03-07-12-13-28-PM.png)

* The numbers represent "annotations", matrix manipulations to propagate these annotations along the graph structure
* We are performing "message passing" in the graph so that we can annotate unknown nodes (the node with the "0")
* We implemented (*almost*) the formula / algorithm shown below, and with [Spektral](https://graphneural.network/layers/convolution/#gcnconv) we can apply it to large-scale graphs
* The formula shows symmetric multiplication (diag(D) on both sides), also with inverse square root (our scaling almost does that)
* Spektral handles the true annotations, represented as one-hot encoded vectors (remember, computers can only understand numbers!)
