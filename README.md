### HDFS putx command

Hao Wang <[haowang@ece.utoronto.ca](mailto:haowang@ece.utoronto.ca)>

---

For some reason, I have to specify the data storage location in HDFS. I checked
this [post](https://stackoverflow.com/questions/32779439/how-to-let-the-hdfss-replica-blocks-position-be-set-by-myself)
and this [patch](https://issues.apache.org/jira/browse/HDFS-2576).

I have added a command named `putx` to the DFS command family. It's a variant of 
the command `put`.

#### Compile

Install dependencies

```shell
sudo apt-get install libprotobuf-dev libprotoc-dev libprotobuf-dev \
protobuf-c-compiler make cmake gcc g++ zlib1g-dev libssl-dev build-essential \
libglib2.0-dev libxrender-dev  libxtst-dev libxtst6
```

```shell
wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
tar xzvf protobuf-2.5.0.tar.gz
cd  protobuf-2.5.0
./configure
make
make check
sudo make install
sudo ldconfig
protoc --version
```

```shell
mvn package -Pdist,native -Psrc -Dtar -DskipTests
```


#### Config

HDFS-6133 provides the ability to exclude favored-nodes (pinned) blocks from the 
HDFS load balancer, by setting the `dfs.datanode.block-pinning.enabled` property to 
`true` in the HDFS service configuration.

This setting prevents HDFS load balancer moving your data to other nodes.

#### Usage

```shell
hdfs dfs -putx -f src dst favored_nodes
```

Examples:

Using hostname
```shell
❯ hdfs dfs -putx -f test2 / hao-ml-1
writeStreamToFile [hao-ml-1:50010]
DFSOutput favoredNodes[hao-ml-1:50010]
DFSOutput nodes: hao-ml-1:50010(10.12.3.40:50010)
```

or Using IP address:

```shell
❯ hdfs dfs -putx -f test2 / 10.12.3.38
writeStreamToFile [/10.12.3.38:50010]
DFSOutput favoredNodes[10.12.3.38:50010]
DFSOutput nodes: hao-ml-3:50010(10.12.3.38:50010)
```

`favoredNodes` is your input. `nodes` are generated by namenode target choosing 
policy. So there's no guarantee that the `favoredNodes` will be enforced.

However, you could repeat `putx` with `-f` until it succeeds placing the data on 
the node you prefer. 

#### Todo

- [x] replace the `nodes` with the `favoredNodes` to implement one-shot enforcement. 
This needs some efforts on HDFS block placement policy


#### Notes

`DFSOutputStream` passes `favoredNodes` to `namenode`:

```java
dfsClient.namenode.addBlock(src, dfsClient.clientName,
                block, excludedNodes, fileId, favoredNodes);
```

`namenode` parses the `favoredNodes` and finds out the placement policy. 

```java
//BlockManager.java
List<DatanodeDescriptor> favoredDatanodeDescriptors = 
    getDatanodeDescriptors(favoredNodes);
final BlockStoragePolicy storagePolicy = storagePolicySuite.getPolicy(storagePolicyID);
final DatanodeStorageInfo[] targets = blockplacement.chooseTarget(src,
    numOfReplicas, client, excludedNodes, blocksize, 
    favoredDatanodeDescriptors, storagePolicy);
```


<p align="right">
<img src="http://www.haow.ca/images/wh_c.png" width=80px" />
</p>
