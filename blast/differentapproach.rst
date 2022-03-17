A Different Approach to Entity Resolution
+++++++++++++++++++++++++++++++++++++++++

What's BLAST?
-------------

There's a possibility that whoever has heard of SparkER was because of BLAST.
We'll keep it simple, as there's a reference of the :doc:`BLAST paper </gettingstarted/extref>` from UNIMORE.
**BLAST** is an acronym for Blocking with Loosely-Aware Schema Techniques and
can be seen (in a really minimalistic way) as an enhanced blocking technique.
Basically BLAST is based on the intuition that similar attributes must have similar 
values, and uses it to exploit the attributes similarity to drastically improve 
recall and precision of the blocking process.
Results have shown that BLAST outperforms some of the traditional meta-blocking
techniques, so you might consider to learn how to use it.

Clusters and Entropy
--------------------

As we stated previously, similar attributes have similar values, and BLAST process
groups these attributes in **clusters** and uses clusters to create blocks in a more efficient
and precise way. 
BLAST also uses the **entropy** of the attributes (Shannon's entropy) to measure the entropy 
of a cluster; the weight associated to each cluster is computed as the average
of the entropies of the contained attributes.
High entropy means that the values of the attributes inside the
cluster are highly heterogeneous, while low entropy means
that the values are very similar.
A match inside a cluster with high entropy has more value than
one inside a cluster with low entropy.





