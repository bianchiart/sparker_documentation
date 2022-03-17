Meta-Blocking
+++++++++++++

Structures for meta-blocking
----------------------------

Before actually perform meta-blocking, we need to create some structures.
These steps use methods from PySpark , we are not going to explain them and 
we highly suggest to read `Spark Documentation <https://spark.apache.org/docs/latest/>`_ to fully understand what's happening.
(in particular the use of map/reduce and SparkContext).

.. code-block:: 

    block_index_map = blocks_after_filtering.map(lambda b : (b.block_id, b.profiles)).collectAsMap()
    block_index = sc.broadcast(block_index_map)

    # This is only needed for certain weight measures
    profile_blocks_size_index = sc.broadcast(profile_blocks_filtered.map(lambda pb : (pb.profile_id, len(pb.blocks))).collectAsMap())

    # Broadcasted groundtruth (only if gt is provided)
    gt_broadcast = sc.broadcast(new_gt)

Perform Meta-Blocking
---------------------

Meta-blocking can be used to further refine the blocks obtained in the previous 
steps, to get a better recall and precision for blocks.
SparkER offers six different strategies for standard meta-blocking:

* (Reciprocal) weighted node pruning;
* Weighted edge pruning;
* (Reciprocal) cardinality node pruning;
* Cardinality edge pruning.

What we said for different blocking strategies, the same applies here; 
different strategies can lead to different performances depending on
various reasons, we leave up to you the choice.

.. code-block:: 
    
    #WNP
    results = sparker.WNP.wnp(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          comparison_type=sparker.ComparisonTypes.OR
                         )
    
    #RWNP
    results = sparker.WNP.wnp(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          comparison_type=sparker.ComparisonTypes.AND
                         )
    
    #WEP 
    results = sparker.WEP.wep(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index
                         )

    #CNP 
    results = sparker.CNP.cnp(
                          blocks_after_filtering,
                          profiles.count(),
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          comparison_type=sparker.ComparisonTypes.OR
                         )
            
    #RCNP
    results = sparker.CNP.cnp(
                          blocks_after_filtering,
                          profiles.count(),
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          comparison_type=sparker.ComparisonTypes.AND
                         )
    
    #CEP 
    results = sparker.CEP.cep(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index
                         )


Let's explain some of the inputs:

* ``weight_type``, the weighting schema used by the algorithm;
* ``comparison_type``, sets the difference between normal node pruning(OR) and reciprocal NP(AND);

For every partition of the RDD the pruning algorithm returns as output a triplet that contains:

* The number of edges;
* The number of matches (only if the groundtruth is provided);
* The retained edges.




Getting stats and results
-------------------------

Like the blocking step, we can get statistics from the pruning algorithm, but in addition
we can also collect the edges (edges are weighted according to the weight strategy provided to the meta-blocking):

.. code-block:: 
    
    num_edges = results.map(lambda x: x[0]).sum()
    num_matches = results.map(lambda x: x[1]).sum()
    print("Recall", num_matches/len(new_gt))
    print("Precision", num_matches/num_edges)
    print("Number of comparisons",num_edges)

    # Collecting edges, this step can always be done, with or without gt

    edges = results.flatMap(lambda x: x[2])
    edges.take(10)


Example data linkage part 3
---------------------------

.. code-block:: 

    block_index_map = blocks_after_filtering.map(lambda b : (b.block_id, b.profiles)).collectAsMap()
    block_index = sc.broadcast(block_index_map)
    profile_blocks_size_index = sc.broadcast(profile_blocks_filtered.map(lambda pb : (pb.profile_id, len(pb.blocks))).collectAsMap())
    gt_broadcast = sc.broadcast(new_gt)
    results = sparker.WNP.wnp(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CBS,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          comparison_type=sparker.ComparisonTypes.OR
                         )

    edges = results.flatMap(lambda x: x[2])
    edges.take(20)
    
