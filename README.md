### HDFS putx command

For some reason, I have to specify the data storage location in HDFS. I checked
this (post)[https://stackoverflow.com/questions/32779439/how-to-let-the-hdfss-replica-blocks-position-be-set-by-myself] 
and this (patch)[https://issues.apache.org/jira/browse/HDFS-2576].

I have added a command named `putx` to the DFS command family. It's a variant of 
the command `put`.

#### Usage