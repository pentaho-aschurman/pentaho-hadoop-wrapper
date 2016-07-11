Wrapping in one Mapper Blueprint
================================

Wrapping an ETL Task in one mapper is based on \[[Action List Processing in MR Blueprint](#action-list-processing-in-mr---blueprint)\], Please read referenced Blueprint before reading this

A Wrapping process in one mapper is an implementation of the “Action List Processing in MR” but with only one Action Item (one line as input). Based on this, there are some configurations changes we can assume:

-   Most times the “input line” content is not important and just works as a helper to push the process to one mapper with the NLineInputFormater.

-   Most times the process will READ/WRITE with custom components therefore we need to set “mapreduce.task.timeout=0”

-   As Hadoop does not have control of the DATA read or written, it can NOT UNDO execution that does not terminate correctly, therefore we need to set “mapreduce.map.maxattempts=1”. This way, if the ETL job fails, it will not try to execute it again

-   After pushing the Task to Hapoop cluster, waiting to finish execution is optional

NOTE: Use this method only when the ETL process is designed to be executed be one Worker and not in parallel. Benefit of this approach is to use Cluster nodes to do the work, but the work will not be distributed.

Action List Processing in MR - Blueprint
========================================

Action List Processing in Map Reduce is a technique to parallelize processes/tasks on a Hadoop cluster leveraging Hadoop Map Reduce default functionality. This technique is strong when the action to be performed requires more computation activity per List Item (line).

To make this work we will use the NLineInputFormat. This input format receives a parameter “mapreduce.input.lineinputformat.linespermap” with the number of fixed lines that will be sent to each mapper. The NLineInputFormat library will see the number of lines in the input file and queue all the chunks of data (lines per mapper) that will be required.
Another interesting aspect of the NLineInputFormat is that the node assigned by the resource manager will be based on available capacity and not data locality.

NOTE: By default, Hadoop monitors if the mappers are READING and/or WRITING data and if no activity is detected in 60 minutes (default wait time) the mapper is terminated.
When we use the wrapper, methodology we could use custom Readers and/or Writers and not the MapReduce Reader/Writers, therefore we may need to add a new parameter called “mapreduce.task.timeout=0” for unlimited wait time
