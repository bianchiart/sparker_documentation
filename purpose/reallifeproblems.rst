Real life example
+++++++++++++++++

A logistic company that works with medical supplies acquires
new customers. These customers already own their product catalogs with
thousands of products. In each catalog the same product has a different identifier,
and a similar but not equal description. The logistic company wants to 
unify the catalogs (i.e. giving a unique id to the same product) 
in order to better organize its warehouse.

.. csv-table:: Catalog A
    :header: PID,Title,Description
    :widths: 5,15,20

    P123X,Syringe 10x10,Syringe 10 ml 10 pack
    P123Y,Syringe 10x100,Syringe 10 ml 100 pack
    P456A,Insulin needle 4x10,Hypodermic insulin needle 4 mm 10 pack
    ...,...,...

.. csv-table:: Catalog B
    :header: ID,Name,Description
    :widths: 5,15,20
    
    1,Syringes 10 ml,Syringe 10 ml x 10 pieces
    2,Syringes 10 ml big pack,Syringe 10 ml x 100 pieces
    3,Small needle,Needle for insulin 4 mm 10 pieces
    ...,...,...

A hypothetic you has to find a solution to this problem and there are two ways to do it.
Let's say you choose the hard way by manually merging the datasets.
Here some reason to say it's a bad idea:

* it requires a lot of people to be efficient, so high effort;
* it requires so a lot of time;
* it's high error prone, since humans will always make some mistakes.

But then you understand that, and choose to use a deduplication tool 
exploiting the descriptions; the pros should be crystal clear, it is faster, accurate 
and combines man work with the tool.

That's where SparkER comes in-handy, it can be used as the tool required for deduplication
purposes, but not only for that.

Now that the introduction has come to an end, it's time to see some code in action.




