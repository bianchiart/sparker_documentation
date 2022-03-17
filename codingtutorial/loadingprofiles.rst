Loading Profiles
++++++++++++++++

But what if you are **not** a casual SparkER user and you want to explore more 
in detail the whole process and the library functions; from now on we will
explain the methods used to perform the actions and how to set custom parameters
and strategies into the methods.

Starting point: wrappers
------------------------

The first thing we have to do is loading the datasets into profiles.
For this purpose, SparkER offers different wrappers; a wrapper is a function that,
given in input the file path of a dataset, it loads it into profiles, which are 
an essential step for the success of the method.
Available wrappers are for ``json`` ``csv`` and ``pandas dataframe``.

.. code-block:: 

    profiles_json=sparker.JSONWrapper.load_profiles(file_path, start_id_from=0, 
                                                    real_id_field="", source_id=0)
    
    profiles_csv=sparker.CSVWrapper.load_profiles(file_path, start_id_from=0, separator=",", 
                                                    header=True, real_id_field="", source_id=0)
    
    profiles_pandas=sparker.PandasWrapper.load_profiles(pandas_df, start_id_from=0, separator=",",
                                                         header=True, real_id_field="", source_id=0)

SparkER offers the possibility to also create pandas dataframes from ``.xslx`` 
files or from ``sql queries`` (supported dbms are ``mysql``, ``postgresql``, ``oracle``, ``mssql``)).

.. code-block:: 
    
    pandas_dataframe_xlsx=sparker.PandasDFMaker.xlsx_df(xlsx_file_path)
    
    pandas_dataframe_query=sparker.PandasDFMaker.dbms_df(host,database,user,password,port, 
                                                            query,dialect=Dialects.POSTGRESQL)

.. caution:: 
    Although pandas is an infinitely flexible library, keep in mind that pandas dataframes
    have a maximum size; so if you're using SparkER on your local machine, we doubt you 
    will reach pandas threshold, but be sure to check `pandas documentation <https://pandas.pydata.org/docs/>`_ to verify 
    it will not run out of memory if you're loading an heavier dataset.

Creating a single profile
--------------------------

This is one of the few step that differentiate deduplication workflow from linkage workflow.
If you're working on a dirty dataset, you already have a single profile, and the only action you need to do
is to extract the max ID from the profiles (that will be used later).

.. code-block:: 
    
    max_profile_id = profiles.map(lambda profile: profile.profile_id).max()


If you are working on clean-clean, then this gets a little trickier.
We're gonna explain with a practical example.

Before that, let's show the groundtruth (optional) methods, that are contained
in json,csv and pandas wrapper classes:

.. code-block:: 
    
    gt = sparker.JSONWrapper.load_groundtruth(groundtruth_fp, id1, id2)



.. note:: 
    The datasets used in the examples are available on SparkER repository
    on GitHub


Example data linkage - Part 1
-----------------------------

.. code-block:: 

    # Profiles contained in the first dataset
    profiles1 = sparker.JSONWrapper.load_profiles('../datasets/clean/abtBuy/dataset1.json', 
                                              real_id_field = "realProfileID")
    # Max profile id in the first dataset, used to separate the profiles in the next phases
    separator_id = profiles1.map(lambda profile: profile.profile_id).max()
    # Separators, used during blocking to understand from which dataset a profile belongs. It is an array because sparkER
    # could work with multiple datasets
    separator_ids = [separator_id]
    profiles2 = sparker.JSONWrapper.load_profiles('../datasets/clean/abtBuy/dataset2.json', 
                                              start_id_from = separator_id+1, 
                                              real_id_field = "realProfileID")
    # Max profile id
    max_profile_id = profiles2.map(lambda profile: profile.profile_id).max()
    profiles = profiles1.union(profiles2)

    #Optional groundtruth step, for these examples we will use groundtruth
    
    # Loads the groundtruth, takes as input the path of the file and the names of the attributes that represent
    # respectively the id of profiles of the first dataset and the id of profiles of the second dataset
    gt = sparker.JSONWrapper.load_groundtruth('../datasets/clean/abtBuy/groundtruth.json', 'id1', 'id2')
    
    # Converts the groundtruth by replacing original IDs with those given by Spark
    new_gt = sparker.Converters.convert_groundtruth(gt, profiles1, profiles2)

As you can see, you will always end with a single set of profiles, you just need to extract the max ID from
both the profiles, and use them to properly link the two. ``separator_ids`` and ``max_profile_id``
will be used in the blocking steps.

Example data deduplication 
-----------------------------------
.. code-block:: 
    
    profiles = sparker.CSVWrapper.load_profiles('../datasets/dirty/cora/cora.csv', real_id_field = "id")
    max_profile_id = profiles.map(lambda profile: profile.profile_id).max()
    gt = sparker.CSVWrapper.load_groundtruth('../datasets/dirty/cora/groundtruth.csv', 'id1', 'id2')
    new_gt = sparker.Converters.convert_groundtruth(gt, profiles)

