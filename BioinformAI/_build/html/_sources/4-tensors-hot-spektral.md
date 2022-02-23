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

---

### How do we code human writen text for machine predictions

* We need to codify the text as [one-hot encoded (1's and 0's) vectors](https://medium.com/analytics-vidhya/one-hot-encoding-of-text-data-in-natural-language-processing-2242fefb2148)

![](https://i.ibb.co/pyG53Dd/Screenshot-2022-02-15-7-36-26-PM.png)

* In the example above there are 7-dimensional vectors, each word in the node annotations represents a separate dimension 
* Annotations can have hundreds or thousands of dimensions, one for each word

---

### An example with simplistic protein annotations

![](https://i.ibb.co/ZSs4NGz/Screenshot-2022-02-19-7-38-54-PM.png)

* Annotations with 5 words, the dictionary provides index for matching the 1's with the set of words
* Tensor: 3-dimensional matrix, (annotation length) x (size of word set) x (number of annotations) 


---
### Our tensor building "algorithm"

* After having our dictionary index in place, go over each annotation sentence
* Split the sentence in words, and for each word get its position on the index
* Remember a tensor is a collection of matrices, each matrix correspond to one annotation
* On each matrix each row designates the position of the word on the index
* On each matrix we have as many rows as there are words (here our annotations are of same length)

---

### Let see it once again

![](https://i.ibb.co/ZSs4NGz/Screenshot-2022-02-19-7-38-54-PM.png)


---
### And now onwards to first building the index for the words

```python

import gensim
from gensim import corpora

annotations = ['This protein phosporylates a gene','This protein catalyzes a reaction','This gene controls RNA expression', 'This gene controls a gene']

annotwords = [[word for word in annotation.split()] for annotation in annotations]
annotdict = corpora.Dictionary(annotwords)

print(annotdict.token2id)

```

* The index is created by genesim.corpora geting the "bag of words" (tokens)
* From the bag of words (if there are replicated words) a unique set is created 
* This dictionary is used as positional reference for the 1's and 0's in the tensor

---
### Now onto building the tensor from the annotations and the index

```python
import numpy as np
annotensor = np.zeros(shape = (len(annotations), len(annotations[0].split(' ')), max(annotdict.token2id.values()) + 1))  

for i, annotation in enumerate(annotations): 
  for j, word in list(enumerate(annotation.split())):
    
    index = annotdict.token2id.get(word)
    annotensor[i, j, index] = 1.   

print(annotations,"\n")
print(annotdict.token2id,"\n")
print(annotensor)
```

* The "enumerate" allows us to have the counter (0,1,2..) and the element we iterate over
* The "i" is for each matrix in the tensor storing one-hot encoding for a single annotation
* The "j" are the rows of the matrix (words in annotation) and "index" the columns position on dictionary

---

### What has been achieved and some notes

* We have encoding the annotations from human language to a standardized machine representation
* The code we seen can be scaled to any size annotations, and thousands of them 
* Notice that annotations were of equal length, not the case with data we seen before
* We either need to fix the data, find different data, or work with "ragged" tensors (possibly difficult)
* We will return to the issue of getting the right data in subsequent sessions

---

### Next: the tip of the tip of the iceberg, for Graph Convolutional Networks

* Graph Convolutional Networks - GCNs, a type of Artificial Neural Networks 
* We will see more Artificial Intelligence and Machine Learning (AI/ML) subsequently 
* Our focus is now to utilize vectorized, one-hot encoded annotation data on each node on our graphs
* We will run a "toy" machine learning algorithm on the graph (prepare from matrix algebra !)

---

### Matrix Algebra refresher

* We discussed [previously how adjacency matrices](https://www.ebi.ac.uk/training/online/courses/network-analysis-of-protein-interaction-data-an-introduction/introduction-to-graph-theory/graph-theory-adjacency-matrices/) represent the graph structure
* Matrix multiplication is row to column, (k,l) x (l,m) = (k,m) inner dimensions must match
* An alternative multiplication representation can be seen [here](https://www.geeksforgeeks.org/wp-content/uploads/strassen_new.png), multiple tutorials on the web 

![width:600](https://miro.medium.com/max/1400/1*YGcMQSr0ge_DGn96WnEkZw.png)

---

### Tensor operations with numpy.array

```python
print(type(annotensor),"\n")
print(annotensor.shape,"\n")

sliceanot = np.sum(annotensor,axis=1)
print(sliceanot,"\n")

print(sliceanot[0:4, 0:5],"\n")

anot = np.sum(sliceanot[0:4, 0:5],axis=1)
anotvec = np.reshape(anot,[4,1])

print(anot,"\n")
print(anotvec,"\n")
```

* Our annotensor variable is a [numpy array](https://numpy.org/doc/stable/reference/generated/numpy.sum.html)
* Crash course in numpy array slicing [here](https://www.earthdatascience.org/courses/intro-to-earth-data-science/scientific-data-structures-python/numpy-arrays/indexing-slicing-numpy-arrays/) and [here](https://www.pythoninformer.com/python-libraries/numpy/index-and-slice/)

---

* We created a compressed representation of example annotations and small tensors 
* Full scale annotations and tensors (with 1000's of entries) in real application 
* We naively represent each node feature (the 1's and 0's for the annotation encoding) as single number
* We will demonstrate matrix operations for Graph Convolutions, following this [example tutorial](https://towardsdatascience.com/how-to-do-deep-learning-on-graphs-with-graph-convolutional-networks-7d2250723780) 
* Eventually we will use [Spectal](https://graphneural.network/) with large scale graphs and data

![height:300](https://i.ibb.co/H2pq4sC/imgonline-com-ua-twotoone-KPNPbw-UW6-MEi-T.jpg)


---
### A graph with the compressed annotations as node attributes

```python
import networkx as nx

A = np.matrix([
    [0, 1, 0, 0],
    [0, 0, 0, 1], 
    [0, 1, 0, 0],
    [1, 0, 1, 0]],
    dtype=float)

G = nx.convert_matrix.from_numpy_matrix(A,create_using=nx.DiGraph)

for i in G.nodes:
	G.nodes[i]["annotval"] = (int(anotvec[i]),'ID=' + str(i))

labels = nx.get_node_attributes(G, 'annotval') 
nx.draw_shell(G, arrows=True, labels=labels)
```
* Again, please refer to [NetworkX documentation](https://networkx.org/documentation/stable/index.html) for the functions we use here

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

* This is the basis for the message passing algorithm in Graph Convolutional neural Networks (GCNs)
* Information propagation based on the structure of the graph, notice that after update we have sum of neighboors 

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

* With addition of the identity matrix we add self-loops in the graph
* This allow us to sum up also the self-value of each node

---
### A summary and where we are going next

* We have seen how to one-hot encode natural language annotations
* Using over-simplified annotation vectors and added them as "features" on nodes of the graph
* Then with a simple matrix multiplication how to propagate these features along the graph structure
* A couple more matrix / linear algebra tricks will give us the formula / algorithm below
* And next will be this algorithm in [Spektral](https://graphneural.network/layers/convolution/#gcnconv) where we use the full one-hot vectors and large-scale graphs

![](https://i.ibb.co/Qf7DtC2/Screenshot-2022-02-22-10-42-34-PM.png)
