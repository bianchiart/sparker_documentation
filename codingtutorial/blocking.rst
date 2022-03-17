Blocking 
++++++++

What's a block?
---------------

A **block** is a group of entity profiles that share the same blocking
key, which key is extracted from a set of keys defined for every profile.
We will not go deeper in the methods for choosing the keys, since there is 
a long article about that in the :doc:`reference section </gettingstarted/extref>`.
What we can say about blocking is that's the most important step for our
purposes. Entity resolution is an inherently O(n^2) problem, and you can 
understand that, if we are working with peta/exabytes of data, ER doesn't scale
very well. But if we can only reduce the comparisons for the number of profiles
in every block, then the comparisons number is much much lower, and since 
comparing can't take forever since "time is money", blocking is a really useful
tool.


Creating blocks
---------------

SparkER offers different techniques for blocks creation, 
we're going to show them all, so you can use which ever you prefer;
they may differ in performance, but that's up to the dataset composition or
other factors.


.. code-block:: 

    blocks = sparker.Blocking.create_blocks(profiles, separator_ids=None, keys_to_exclude=None,
                      attributes_to_exclude=None,
                      blocking_method=sparker.BlockingKeysStrategies.token_blocking
                      )

The function above takes as input the set of profiles obtained as a result of the
loading profiles step, the separator ids (you may or may not have to input it, depends if you are 
doing deduplication or linkage),some optional parameters to filter keys or attributes you want to
bypass and returns a set of blocks, containing profiles that share the same keys.
This is a standard token blocking strategy, is general purpose and the easiest and quickest to use,
but the *blocking_method* parameter is the one that you can change to use a different blocking strategy.

Other strategies are:

* ``BlockingKeysStrategies.token_blocking_w_attr``
* ``BlockingKeysStrategies.ngrams_blocking``

While both strategies share the same parameters as the above method, n-gram blocking actually require an 
additional parameter ``ngram_size``, which gives the size of the n-grams used for the strategy, by default 
it's set to 3.

To check the number of blocks created you can simply do:

.. code-block:: 

    print("Number of blocks",blocks.count())

Block Purging and Cleaning
--------------------------

The next step is to filter those blocks.
SparkER implements two block filtering methods:

* **Block purging**: discard the largest blocks that involve too many comparisons;
* **Block cleaning**: removes for every profile the largest blocks in which it appears.

For the coding part:

.. code-block:: 

    #Block purging
    blocks_purged=block_purging(blocks, smooth_factor)

    #Block cleaning
    (profile_blocks, profile_blocks_filtered, blocks_after_filter_filtering) = block_filtering_quick(blocks, r, separator_ids=None)

For the block purging method, takes in input the blocks created in the previous step, and 
a smooth factor that must be >= 1, a lower value means a more aggressive purging. It returns
the remaining blocks after the discard.
For the block cleaning method, it takes in input the block set after purging, a parameter 
in range ]0, 1[ , a lower value mean a more aggressive cleaning and the separator ids (same criteria as blocks creation)
and returns a triplet containing:

* ``profile_blocks``, the RDD of Profile_blocks calculated from blocks;
* ``profile_blocks_filtered``, the RDD of filtered Profile_blocks;
* ``blocks_after_filter_filtering``, the RDD of blocks generated from profile_blocks_filtered.

Checking performances (Optional)
--------------------------------

If the groundtruth is provided, after every blocking step you
can check recall, precision and number of comparisons.

.. code-block:: 

    recall, precision, cmp_n = sparker.Utils.get_statistics(blocks_after_filtering, max_profile_id, new_gt, separator_ids)

    print("Recall", recall)
    print("Precision", precision)
    print("Number of comparisons", cmp_n)

Again, is not a necessary step, but is useful to have some checking on the process.
We're not going to explain the parameters since we have already encountered all of them
nor what it does, since is pretty self-explainatory.

Example data linkage - Part 2
-----------------------------

.. code-block:: 

    # Let's use n-grams blocking strategy
    blocks = sparker.Blocking.create_blocks(profiles, separator_ids, 
                                        blocking_method=sparker.BlockingKeysStrategies.ngrams_blocking,
                                        ngram_size=4)

    blocks_purged = sparker.BlockPurging.block_purging(blocks, 1.025)
    (profile_blocks, profile_blocks_filtered, blocks_after_filtering) = sparker.BlockFiltering.block_filtering_quick(blocks_purged, 0.8, separator_ids)
    
    # Since in part 1 we loaded groundtruth, let's get the statistics.
    recall, precision, cmp_n = sparker.Utils.get_statistics(blocks_after_filtering, max_profile_id, new_gt, separator_ids)

    print("Recall", recall)
    print("Precision", precision)
    print("Number of comparisons", cmp_n)

Since the only difference from blocking and beyond is the presence or not of separator_ids, 
we will not continue with the data deduplication example, since it uses the same exact methods.