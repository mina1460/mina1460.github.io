---
title: "Approximate_nearest_neighbor"
date: 2022-05-02T17:05:48+02:00
tags: [datascience, KNN, ANN]
---

# Approximate Nearest Neighbors

## problem with KNNs 
Finding the nearest neighbors in your data can be a very useful exercise in hundreds of applications. You will always find KNN models making their way into your solution in one way or another. They are used in recommendation engines, anomaly detection, classification of semantically similar documents, etc, but they have a serious problem, which is scalability. 

KNN models are lazy non-parametric models, which means that when you do the famous model.fit(), they don’t actually learn anything, they just store a reference for the training data that they will use in doing the search.
So far so good, but why is scalability an issue here? Simply because the algorithmic complexity of running a single query is Linear, O(N). Thus, if you have N users for whom you want to find their nearest neighbors, the complexity will be quadratic, which is not really an option in the big data era. Note that we assumed that the distance measure you are using is running in constant time O(1), which is rarely the case with high-dimensional data. 

To make sure you understand the last part, let’s say that you are using cosine similarity to measure the distance between two items in a dataset, and if each item is represented as a high dimensional vector of (100+ entries), you will compute the dot product between each pair of vectors each time, which adds another layer of complexity.

![cosine similarity](https://media-exp1.licdn.com/dms/image/C4D12AQHnPa6XJwklBw/article-inline_image-shrink_1500_2232/0/1651434418634?e=2147483647&v=beta&t=ghc0xBSuoR2RESJpSzOCekgov_y2pr8spWrLS-JnMQA)


Imagine Youtube running this algorithm for their 2.6 billion active users, and then wait for a few years until you get 1 recommendation that you may or may not like 😈.

![84](https://media-exp1.licdn.com/dms/image/C4D12AQHDT4U5yhz4EQ/article-inline_image-shrink_1500_2232/0/1651434573950?e=2147483647&v=beta&t=OOX_zNPghGAKim-Du0-At2jjRPV3vziY7FT0sQdLprs)

## Solution ?

 I think you now understand why it is usually a bad idea to use KNN in production systems, but surely there should be a solution to this, right? Yes, here is where we use the approximate nearest neighbors search (ANN), which can execute a single query in O(log N) complexity. This is a HUUUUGE difference, even if it sacrifices the accuracy a little bit. KNN was not really an option to consider. There are a couple of implementations for ANN, including Facebook’s PySparseNN (for sparse interaction matrices), annoy (Approximate nearest neighbor, Oh YEAH) by Spotify, etc.

here is a visualization for how Annoy works.

![img](https://media-exp1.licdn.com/dms/image/C4D12AQH27SvTXzNxvg/article-inline_image-shrink_1500_2232/0/1651434527947?e=2147483647&v=beta&t=sSZCZO1GLuEWcvXbOkfLRK376-wU-IOORxkRCw-tsW4)

## How does it work?
The idea is very simple, you pick two points from your data randomly, and find a hyperplane that is equidistant between the two points to split the data. Then find two other points, and repeat, until you end up with a division that has “K” number of points, then you stop.

![how](https://media-exp1.licdn.com/dms/image/C4D12AQEoMfFf51D7KA/article-inline_image-shrink_1500_2232/0/1651434831833?e=2147483647&v=beta&t=nVkWExjDJIFwB5wqtEodFBonDLHT0MMBY4DSCaoxFuI)

Now, you have a tree that you can simply traverse to find a group of neighbors to your query point and get your results in Logarithmic complexity. You can make it even better by not relying on a single tree, but a whole forest of trees to make the splits. Just keep in mind that these trees are stored in memory, and for big datasets, it is usually very heavy, and the complexity would increase you now have to traverse multiple trees to make your query. 

![GIF](https://media-exp1.licdn.com/dms/image/C4D12AQHDNxPaOVnV1g/article-inline_image-shrink_1000_1488/0/1651434672506?e=2147483647&v=beta&t=HPmSo3fVQBnQ1__5bpHBkBw4gwMUh6WWad6ayiIfOS0)

and here is what it looks like at making your queries after you have built your forest

![final](https://media-exp1.licdn.com/dms/image/C4D12AQEQfUSZiE728A/article-inline_image-shrink_1000_1488/0/1651435204463?e=2147483647&v=beta&t=uJyDu7sCqalE_wTlaptaBJMlODK_CzFwPwuoBKZNyFs)

Now you can simply run your distance function on comparison on the few nodes that the tree traversal led to, which makes this system scalable while providing good results.

## Sources 
source of the amazing images used in this article: [link](https://erikbern.com/2015/10/01/nearest-neighbors-and-vector-models-part-2-how-to-search-in-high-dimensional-spaces.html)
