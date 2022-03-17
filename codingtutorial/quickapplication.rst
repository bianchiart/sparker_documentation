Quick Application
+++++++++++++++++

Profiles, clean and dirty datasets
----------------------------------

Before moving on, we need to understand what's a profile and the difference between
clean and dirty dataset.
A **profile** is a uniquely identified set of name-value pairs that corresponds to a
single real world object.
**Clean** datasets are datasets that do not contain any duplicates,
on the other hand **dirty** datasets are the ones that contain duplicates...
who would have imagined that.
We can already see that data deduplication is only necessary for the dirty ones.
Anyway, differenciating between them is essential for some of SparkER methods
to work properly.

We can finally begin to code.
First of all, you need to import the library.

.. code-block:: 
    
    import sparker



Now we can see some high level code, these methods we're about to show
are facades, meaning you give in input the files and it outputs the results.

Quick data deduplication
------------------------

.. code-block:: 
    
    profiles, results, stats = sparker.QuickMetaBlocking.data_deduplication(profiles_fp,groundtruth_fp=None,
                                                                    real_id_f="id",file_extension="csv",
                                                                    weighting_type=WeightTypes.CBS)

Given a (dirty) dataset file path, the real ID field, which is the name of the attribute 
that contains the profile identifier in the file, and the file_extension (csv by default),
it returns a set of profiles, a list containing the number of matches, the number of edges and the number of comparisons
and a dictionary containing the statistics of the final meta-blocking.

``results = [num_matches, num_edges, comparisons]``

``stats = [recall, precision, comparisons]``

We will ignore for now the weighting_type, we'll talk more about that :doc:`here </codingtutorial/blocking>`;
the groundtruth is an optional parameter, is a dataset that contains the id correspondence 
of the profiles, so that SparkER can analyze the process performances.

.. caution:: 
    Although is not a necessary parameter for the function to work properly,
    it's worth to mention that if the groundtruth is not provided SparkER can't 
    measure the number of matches of the results, recall and precision of the stats,
    and those stats will be left empty.


Quick data linkage
----------------------------

.. code-block:: 
    
    profiles, results, stats = sparker.QuickMetaBlocking.standard_metablocking(profiles1_fp,profiles2_fp,
                                                                    groundtruth_fp=None,real_id_f="id",
                                                                    file_extension="json", weighting_type=WeightTypes.CBS,
                                                                    mb_schema="wnp",comp_type=ComparisonTypes.AND)

As you can see, some parameters are the same as the above function, 
you can logically guess that profiles2_fp has the same function as profiles1_fp.
The final 3 parameters will be explained more in details later.
Given the parameters, it returns the same results as the data deduplication function above.
This function actually performs a different task from the other one, in fact
you use this function when you have two clean datasets and you want to discover 
the duplicates between them. This operation is called **data linkage**.

There is another quick method for BLAST meta-blocking, but for BLAST there
is a dedicated section, so if you want to skip to that you can click here.

