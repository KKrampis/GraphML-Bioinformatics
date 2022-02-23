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

### Graphs and and machine learning for protein function prediction 

* Building the graph instance from the experimental data (i.e. PPI network if protein interactions)
* Loading the graph entities (nodes mainly) with multidimensional data in order to enable predictions
* Running machine learning algorithms on the graph for protein function inference on the unknown nodes

---

### Prepare file with network data 

```python

!wc -l 15761153-cut.tsv
!cut -f 1-2,17-22 ./content/15761153.txt | head -n 2
!cut -f 1-2,19-20./content/15761153.txt > ./content/15761153-cut.tsv

```

---

### Python list slicing and lambda functions

* Multiple tutorials can found online on these topics, example links are given here
* On lambdas, see [here](https://cs.stanford.edu/people/nick/py/python-map-lambda.html) and [here](https://www.geeksforgeeks.org/python-lambda-anonymous-functions-filter-map-reduce/)
* For list slicing see [here](https://www.geeksforgeeks.org/python-list-slicing/) and [here](https://stackoverflow.com/questions/509211/understanding-slice-notation)
* Read material on links above !

---
### Let's parse out the file and make a Graph with NetworkX

```python

with open('/home/bioitx/Author/private-bioitx-AI/BioinformAI/content/15761153-cut.tsv', 'r') as f:
  graphdata = [line.rstrip().split('\t') for line in f]

print(graphdata[0:3])

graphdataclean = []

for row in graphdata[1:]:
  graphdataclean.append(list(map(lambda element: element[10:], row[0:2])) \
                      + list(map(lambda element: element[17:21], row[2:])))

print("\n\n")
print(graphdataclean[0])

```

---

### Let's parse out the file and make a Graph with NetworkX

```python

nodes = []

for row in graphdataclean:
	nodes.append( (row[0],{'type' : row[2]}) )
	nodes.append( (row[1],{'type' : row[3]}) )

edges = []
for row in graphdataclean:
	edges.append( (row[0],row[1]) )

import networkx as nx
G = nx.Graph()
G.add_nodes_from(nodes)
G.add_edges_from(edges)
labels = nx.get_node_attributes(G, 'type') 
nx.draw_networkx(G, labels=labels)

```

* First we will make a list / dictionary with the node attributes (the prey-bait)
* Then an array with the edges. Refer to Networkx [documentation](https://networkx.org/documentation/stable/tutorial.html) for [example](https://i.ibb.co/0C0XZ48/Screenshot-2022-02-14-9-44-36-PM.png)

---

### A smaller graph to visualize easier

* Possibly some artefacts of nodes missing labels due to the slicing below
* Open question : how does "prey-bait" node connections in graph correspond to experimental design

```python

nodesub = nodes[1:50]
edgesub = edges[1:50]

subG = nx.Graph()
subG.add_nodes_from(nodesub)
subG.add_edges_from(edgesub)
labels = nx.get_node_attributes(subG, 'type') 
nx.draw_networkx(subG, labels=labels)

```
---

### More dimensions in the node data for machine predictions

![](https://i.ibb.co/vLbRWPj/Screenshot-2022-02-15-7-17-28-PM.png)

* Each node has a single parameter (a single dimension) of data associated with it (prey or bait)
* The more data, the better for machine learning predictions of unknown nodes
* The annotations field has complex function description entered by bioinformatics analysts / curators

---

### How do we code human writen text for machine predictions

* We need to codify the text as [one-hot encoded (1's and 0's) vectors](https://medium.com/analytics-vidhya/one-hot-encoding-of-text-data-in-natural-language-processing-2242fefb2148)

![](https://i.ibb.co/pyG53Dd/Screenshot-2022-02-15-7-36-26-PM.png)

* In the example above there are 7-dimensional vectors, each word in the node annotations represents a separate dimension 
* Annotations can have hundreds or thousands of dimensions, one for each word

---

### How do we code human writen text for machine predictions

* Capturing in the vectors the context (surrounding) of each word for defining meaning precisely
* [Natural Languange Processing](https://en.wikipedia.org/wiki/Natural_language_processing) (NLP) for human language data mining
* For example: **"We went to the river bank"**
* **"We made a deposit at the bank"**
* Difference in meaning of word "bank" needs to be modeled in data for machine learning / AI applications

---

### Back to bioinformatics !

* Our focus is how to add the rich annotation data on each node on our graphs
* Add in a vectorized, one-hot encoded format data on each node, not focus on NLP
* We will use algorithms and Python libraries such as [Word2vec](https://towardsdatascience.com/a-word2vec-implementation-using-numpy-and-python-d256cf0e5f28) also [here](https://stackabuse.com/implementing-word2vec-with-gensim-library-in-python/) and [Gensim](https://www.machinelearningplus.com/nlp/gensim-tutorial/) from NLP
* Once the graph is in place and one-hot encoded data on the nodes, we will run machine learning algorithms on it
* Please read carefully the content from the links above !

---

### Bag of words representation

* Create a dictionary - the set of unique words - from the sentences (or the "corpus")
* Parse the sentences and create a one-hot encoded vector from each sentence:

![](https://miro.medium.com/max/1400/1*hLvya7MXjsSc3NS2SoLMEg.png)
[Image credit](https://www.ronaldjamesgroup.com/blog/grab-your-wine-its-time-to-demystify-ml-and-nlp)

* Easy to implement, but a lot of wasted space, context not differentiated ("bank deposit" vs "river bank")

---

### Bag of words representation
W
* Create a dictionary - the set of unique word counts - from the annotations (or the "corpus")
* Then  create a one-hot encoded group of vectors - multidimensional matrix (or tensor) attached to each node
* First lets have our files with protein ids and annotations as accompanying data

```python
!cut -f 1-2,26-27 ./content/15761153.txt | head -n 2
!cut -f 1-2,26-27 ./content/15761153.txt > ./content/15761153-cut.tsv
```

---
### Let's parse out the annotations

```python

with open('/home/bioitx/Author/private-bioitx-AI/BioinformAI/content/15761153-cut.tsv', 'r') as f:
  graphdata = [line.rstrip().split('\t') for line in f]

graphdataclean = []

for row in graphdata[1:]:
  graphdataclean.append(list(map(lambda element: element[10:], row[0:2])) \
                      + list(map(lambda element: element[22:], row[2:])))

print(graphdataclean[0])

```

---
### Let's one-hot encode the annotations

* Do not forget to run "!pip install gensim"
* We will follow examples from [this tutorial](https://www.machinelearningplus.com/nlp/gensim-tutorial/)

```python

import gensim
from gensim import corpora

corpus=[]
for row in graphdataclean:
	corpus.append(row[2])
	corpus.append(row[3])

  #print(corpus[0:2])

annotwords = [[word for word in annotation.split()] for annotation in corpus]
annotdict = corpora.Dictionary(annotwords)

#print(annotwords[0:2])
print(annotdict.token2id)

```


```python

print(annotdict[100])

```
