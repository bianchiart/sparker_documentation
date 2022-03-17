How to use BLAST
++++++++++++++++


Quick BLAST method
------------------
Like for the data deduplication and data linkage, we have also implemented
a method that wraps up the entire BLAST process in one single facade function:

.. code-block:: 

    profiles, results, stats = sparker.QuickMetaBlocking.blast(profiles1_fp,profiles2_fp, 
                                                            groundtruth_fp=None,real_id_f="id",
                                                            file_extension="json", chi_divider=2.0)

Works exactly like data linkage method seen :doc:`here </codingtutorial/quickapplication>`, but uses BLAST workflow and let's the user 
set the ``chi_divider`` value. 
We will now get into more detail.


Attribute Clustering
--------------------
It's important to point out that BLAST can only be used with clean-clean
datasets. 
While BLAST sounds much much cooler than standard blocking/meta-blocking,
the actual coding part is really similar to the data linkage workflow.
So you want to begin by loading the profiles.
After that, here's where the first difference, or addition we might say, occurs:

.. code-block:: 
     
    clusters = sparker.AttributeClustering.cluster_similar_attributes(profiles, num_hashes, 
                                                    target_threshold, compute_entropy=False)
    

Given as input:

* ``profiles``,self-explanatory;
* ``num_hashes``, number of hashes to use for LSH, since BLAST employs ,ocal sensitivity hashing to automatically align the attributes;
* ``compute_entropy``, if set to True it computes the entropy of each cluster of attributes that can be used to further improve the meta-blocking performance;
* ``target_threshold``,  regulates the similarity that the values of the attributes should have to be clustered together;

it return a list of clusters, containing similar attributes.

``clusters`` will then be taken as input from the function ``create_blocks_clusters`` for the blocking process.

.. code-block:: 

    blocks = sparker.Blocking.create_blocks_clusters(profiles, clusters, separator_ids)


Using Entropy and Chi-Square Divider
------------------------------------

Entropy computing is an additional step that we must take care of when create structures before meta-blocking.

.. code-block:: 

    block_entropies = sc.broadcast(blocks.map(lambda b : (b.block_id, b.entropy)).collectAsMap())



Now everything is set to use meta-blocking.

.. code-block:: 
    
    results = sparker.WNP.wnp(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CHI_SQUARE,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          use_entropy=True,
                          blocks_entropies=block_entropies,
                          chi2divider=2.0
                         )

If you have seen the standard meta-blocking explanation, you can already see
that there are some more parameters in input:

* ``weight_type``, must be set to ``CHI_SQUARE``, since BLAST uses that weight measure;
* ``use_entropy``, must be set to ``TRUE``, for obvious reasons;
* ``blocks_entropies``, self-explanatory;
* ``chi2divider``,  regulates the pruning aggressivity, a lower value performs a more aggressive pruning.

Everything else is just as you would normally do with the standard workflow.


BLAST complete example
----------------------

.. code-block:: 

    # Load profiles
    profiles1 = sparker.JSONWrapper.load_profiles('../datasets/clean/DblpAcm/dataset1.json', 
                                              real_id_field = "realProfileID", 
                                              source_id=1)
    separator_id = profiles1.map(lambda profile: profile.profile_id).max()
    separator_ids = [separator_id]
    profiles2 = sparker.JSONWrapper.load_profiles('../datasets/clean/DblpAcm/dataset2.json', 
                                              start_id_from = separator_id+1, 
                                              real_id_field = "realProfileID", 
                                              source_id=2)
    profiles = profiles1.union(profiles2)

    # Load groundtruth (optional, but we can see stats)
    gt = sparker.JSONWrapper.load_groundtruth('../datasets/clean/DblpAcm/groundtruth.json', 'id1', 'id2')
    new_gt = sparker.Converters.convert_groundtruth(gt, profiles1, profiles2)

    # Clusters creation
    clusters = sparker.AttributeClustering.cluster_similar_attributes(profiles,
                                  num_hashes=128,
                                  target_threshold=0.5,
                                  compute_entropy=True)

    # Blocking creation and filtering
    blocks = sparker.Blocking.create_blocks_clusters(profiles, clusters, separator_ids)
    blocks_purged = sparker.BlockPurging.block_purging(blocks, 1.005)
    (profile_blocks, profile_blocks_filtered, blocks_after_filtering) = sparker.BlockFiltering.block_filtering_quick(blocks_purged, 0.8, separator_ids)

    # Structures pre-MB
    block_index_map = blocks_after_filtering.map(lambda b : (b.block_id, b.profiles)).collectAsMap()
    block_index = sc.broadcast(block_index_map)
    block_entropies = sc.broadcast(blocks.map(lambda b : (b.block_id, b.entropy)).collectAsMap())
    profile_blocks_size_index = sc.broadcast(profile_blocks_filtered.map(lambda pb : (pb.profile_id, len(pb.blocks))).collectAsMap())
    
    # Gt broadcasting (only if gt is provided)
    gt_broadcast = sc.broadcast(new_gt)

    # Meta-Blocking
    results = sparker.WNP.wnp(
                          profile_blocks_filtered,
                          block_index,
                          max_profile_id,
                          separator_ids,
                          weight_type=sparker.WeightTypes.CHI_SQUARE,
                          groundtruth=gt_broadcast,
                          profile_blocks_size_index=profile_blocks_size_index,
                          use_entropy=True,
                          blocks_entropies=block_entropies,
                          chi2divider=2.0
                         )

    # Getting the results
    num_edges = results.map(lambda x: x[0]).sum()
    num_matches = results.map(lambda x: x[1]).sum()
    print("Recall", num_matches/len(new_gt))
    print("Precision", num_matches/num_edges)
    print("Number of comparisons",num_edges)
    edges = results.flatMap(lambda x: x[2])



     
