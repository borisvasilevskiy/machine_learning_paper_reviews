# GNNs and graph planarity detection

This note contains an overview of several papers related to planarity detection via GNNs.

## Graph planarity introduction

An undirected graph is called 'planar' if it can be drawn on a plane without edge intersections. There are well-established mathematical criteria for graph planarity. One of them is Kuratowski's criteria, which states that graph planarity is equivalent to the absence of a subgraph that is a subdivision of K_5 (a complete graph on 5 vertices) or K_33 (a complete bipartite graph with 3+3 vertices). A subdivision of a graph results from inserting vertices into edges zero or more times. Details can be found here: https://en.wikipedia.org/wiki/Planar_graph.

There are several planarity detection algorithms that work in linear time O(n), where n is the number of vertices. You may wonder why it's not O(E), where E is the number of edges, since that's the minimum required to read the graph. Well, there's a theorem (https://en.wikipedia.org/wiki/Planar_graph#Other_criteria) that claims the number of edges in a planar graph is <= 3n - 6. This means we won't have to read more than 3n - 6 edges of the graph to establish its planarity.

Let's note that Kuratowski's criteria is almost local. By 'local,' I mean that a constant-length neighborhood of each vertex needs to be examined for the planarity check. By 'almost,' I mean it's not completely local because it has to deal with subdivisions, i.e. smooth all vertices with power=2 from the subgraph.

## Motivation to run GNNs on algorithmically solvable problems

Historically, image classification was solved with ConvNets and other techniques. However, we don't know a lot about how people solve the image classification task, as it's an unconcious mechanism. It makes it harder to understand how NNs do it. People have found common approaches in NNs and the brain, e.g, circuits, however this knowledge will not be complete until brain functioning is understood really well.

In case of algorithmic tasks on graphs we know the solution. If we train NNs to solve these problems, it might be easier to understand how are they learning and what did they learn. It's about teaching NNs what we know rather than what we can unconciously do. Therefore, training NNs algorithms can bring insights that we couldn't obtain when studying image classification, natural language processing.

## Motivation to solve planarity with GNNs

The fact that the task is algorithmically solvable makes it an easy benchmark since the dataset can be auto-generated and can contain any number of graphs with various properties.

The fact that it's almost local makes it a good candidate for GNNs to solve it with high accuracy. For instance, we can generate a dataset to contain only graphs that are solvable with the local criteria. K_5 is located in the 1-hop neighborhood (it's called a '1-disc') of any of its vertices, and K_33 is located in the 2-hop neighborhood. It means that 2 layers of GNN may already be sufficient to learn the local Kuratowski's criteria. A couple more layers won't do any harm, of course, but the point is that GNN has access to all the information needed to solve the task perfectly.

Besides that, it's interesting to learn how GNN would deal with the main task that's not local.

# Literature overview

I haven't found any papers that benchmark different GNN approaches on the graph planarity task. However, I've found several interesting more theoretical papers. Here they are.

## A Property Testing Framework for the Theoretical Expressivity of Graph Kernels
by Nils M. Kriege, Christopher Morris, Anja Rey, Christian Sohler / Proceedings of the Twenty-Seventh International Joint Conference on Artificial Intelligence / Main track. Pages 2348-2354. https://doi.org/10.24963/ijcai.2018/325

A graph kernel is a function that calculates a numeric graph similarity based on two given graphs:

k(G_1, G_2) -> real number >= 0

One connection between graph kernels and GNNs is the following: imagine we have a kernel, then we can employ GNNs to embed a graph into a d-dim space so that the graph similarity provided by the kernel is close to the scalar product of the produced embeddings. That can be done using ML methods, particularly with GNNs.

Different kernels produce different graph embeddings that can be later used to solve graph classification tasks. For instance, one can train a logistic regression that takes graph's embedding an an input and tries to predict graph's planarity. One can go with several FC layers or even with more advanced architecture. In this approach, the graph embedding is generated via fully unsupervised way, and then used as an input feature to a supervised task.

One of the paper's contribution that is of interest here, is that they consider several popular kernels and several popular graph properties (such as planarity) and establish theoretical abilities to determine these properties using the embeddings generated with the kernel.

Here're some of their results (cited from the paper):

>* The random walk kernel and the Weisfeiler–Lehman subtree
> kernel both fail to identify connectivity, planarity, bipartiteness and triangle-freeness (see Theorems 4.1, 4.2).
>* The graphlet kernel can identify triangle-freeness, but fails
>to distinguish any graph property (see Theorem 4.5).
>* We define the k-disc kernel and show that it is able to
>distinguish connectivity, planarity, and triangle-freeness
>(see Theorem 5.4).

Therefore, if one is going to embed a graph using unsupervised learning kernels, a good kernel to start with is 'k-dics'.

## GNNs on NP-hard problems

As a very simple and not accurate explanation, an NP-hard problem is a combinatorial problem which polynomial solution is not yet found. Moreover, there's a mathematical fact that if a solution is found to one NP-hard problem then it will be possible to polynomially solve all other NP-hard problems. Examples: graph isomorphism, minimum dominating set, minimum vertex cover, maximum matching.

GNN's inference takes polynomial time relative to graph's size, GNN can't precisely solve these problems if a widely accepted assumption NP != P is true. However, it's interesting to search for more effective algorithms learned by GNNs - I think that's one of motivations.

To estimate effectiveness of an algorithm solving NP-hard problems there's a notion of approximation ratios. It is actually a success metric. I won't go into detail as I don't know much about it.

### How powerful are graph neural networks?
By Keyulu Xu, Weihua Hu, Jure Leskovec, and Stefanie Jegelka
CoRR, abs/1810.00826, 2018.

### Approximation Ratios of Graph Neural Networks for Combinatorial Problems
by Ryoma Sato, Makoto Yamada, Hisashi Kashima
[arXiv:1905.10261 [cs.LG]](https://arxiv.org/abs/1905.10261)

Authors tell that the motivation of the paper is basically the same as of Xu et al (described above), but they consider different NP-hard problems and more than 1 of them.

The main topic of the paper is GNN performance on following NP-hard problems:
* minimum dominating set problem
* minimum vertex cover problem
* maximum matching problem

It is assumed that graphs have a bounded maximum degree (by some constant that doesn't depend on the size). As node features they mostly use one-hot encoding of node's degree.

They employ a theory of distributed local algorithms that have a lot in common with GNNs. There's actually a simple generalization of GNNs that is equivalent (in graph problem solving ability) to distributed local algorithms. The latter seems to be well-studied. With that help, they show theoretical boundaries on GNN ability to solve aforementioned NP-hard problems.

They generalize GNNs to a newly proposed Consistent Port Numbering GNNs that use port numbering (the paper contains a good definition of it).

Summarizing, GNNs are worse than simple greedy algorithms. However, in case of Minimum Vertex Cover Problem a Consistent Port Numbering GNNs happened to be able to learn a non-trivial algorithm [2]. To my mind, it's very interesting to actually train such a GNN and investigate what did it actually learn.

[2] Matti Åstrand, Patrik Floréen, Valentin Polishchuk, Joel Rybicki, Jukka Suomela, and Jara Uitto. A local 2-approximation algorithm for the vertex cover problem. In Proceedings of 23rd International Symposium on Distributed Computing, DISC 2009, pages 191–205, 2009.

