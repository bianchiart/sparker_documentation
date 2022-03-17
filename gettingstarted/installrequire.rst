Installation and Requirements
+++++++++++++++++++++++++++++

Pre-Requisites
--------------

As previously stated, SparkER is a python library which works under Apache Spark, 
so in order to begin you must download everything needed for Apache's software.
In addition, you will need git to clone SparkER repository to your local machine.

Here is a list of links to get everything you need:

* `Python <https://www.python.org/downloads/>`_
* `Java <https://www.java.com/download/manual.jsp>`_
* `Apache Spark <https://spark.apache.org/downloads.html>`_
* `Git <https://git-scm.com/downloads>`_

.. note:: 
        You can use Jupyter Notebook as your working environment, and
        that's what we recommend since it enables you to observe in an easier way 
        what happens with every step of the workflow.

SparkER installation
--------------------

Now that you have everything you need, you can clone the repository 
of the source on its `repository <https://github.com/Gaglia88/sparker>`_.
After that, installing SparkER is a pretty simple task, and you can 
do the following shell commands:

``cd ../sparker/python``

``pip setup.py install``

If you don't want to install SparkER on your device, you can simply install the dependencies
and use SparkER directly from your main folder.

``pip install -r requirements.txt``