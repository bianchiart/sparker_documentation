Understanding the Purpose
+++++++++++++++++++++++++
Why SparkER exists?
-------------------

If you have already skipped to the coding parts (i know you did -_-) and 
you came back here, you may be wondering why would you ever use this library.
In that case, let me briefly introduce you to some **big** concepts that summarize 
the point of the library.

Entity Resolution
-----------------

Entity Resolution is the task of finding every instance of an entity, 
such as a customer, across all enterprise systems, applications, 
and knowledge bases on-premises and in the cloud.
In other words, if you search for the example entity "Monkey D. Luffy", you want
to get **every single** instance of that entity, instances that can be found
in different aspects, can be in an enterprise database or in a 
website, saved in different shapes and, if you are unlucky and in most cases you 
will be, it may be saved with errors; you are looking for "Monkey D. Luffy" and instead
in a database it's saved as "Monkey D. Loofy" because of writing mistake.
ER is a heavy task for engineers and programmers since the amount of data 
is growing really fast.

Data Deduplication
------------------

To make it really simple, data deduplication is a process that eliminates 
excessive copies of data and significantly decreases storage capacity requirements.
Let's suppose you have an enormous dirty dataset, with lots of duplicates between entities.
Let's say that someone really didn't pay attention to the increasing number of duplicates
and now you are stucked with this pile of useless duplicates. There's the
need to get rid of the duplicates because not only they stink so bad, but  
reading the database requires a lot of time, and in the modern society wasting time
is not an option.
Data deduplication techniques can get rid of duplicates by using some algorithms and 
mathematical 'tricks'.

In the next page, we'll give you a real life scenario to which SparkER is applicable.

