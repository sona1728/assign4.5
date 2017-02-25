1) What is Checksum? 

ans:
- Checksums are used to ensure the integrity of a file after it has been transmitted from one storage device to another.
- This can be across the Internet or simply between two computers on the same network. Either way, if you want to ensure that the transmitted file is exactly the same as the source file, you can use a checksum. 
- Importance To check the data integrity of the stored data. To have the data transferred or read completely without any voids and breaks in between to avaoid data corruption Checksum in hadoop The HDFS client software implements checksum checker.
- When a client creates an HDFS file, it computes a checksum of each block of the file and stores these checksums in a separate hidden file in the same HDFS namespace. When a client retrieves file contents, it verifies that the data it received from each DataNode matches the checksum stored in the associated checksum file. If not, then the client can opt to retrieve that block from another DataNode that has a replica of that block.
- If checksum of another Data node block matches with checksum of hidden file, system will serve these data blocks.

2) 
#ANATOMY TO FILE WRITE TO HDFS:

- The client creates the file by calling create() on DistributedFileSystem. DistributedFileSystem makes an RPC call to the namenode to create a new file in the filesystem’s namespace, with no blocks associated with it.
- The namenode performs various checks to make sure the file doesn’t already exist, and that the client has the right permissions to create the file. If these checks pass, the namenode makes a record of the new file; otherwise, file creation fails and the client is thrown an IOException. The DistributedFileSystem returns an FSDataOutputStream for the client to start writing data to. Just as in the read case, FSDataOutputStream wraps a DFSOutput Stream, which handles communication with the datanodes and namenode. 
- As the client writes data, DFSOutputStream splits it into packets, which it writes to an internal queue, called the data queue. The data queue is consumed by the Data Streamer, whose responsibility it is to ask the namenode to allocate new blocks by picking a list of suitable datanodes to store the replicas. 
- The list of datanodes forms a pipeline—we’ll assume the replication level is three, so there are three nodes in the pipeline. The DataStreamer streams the packets to the first datanode in the pipeline, which stores the packet and forwards it to the second datanode in the pipeline. 
- Similarly, the second datanode stores the packet and forwards it to the third (and last) datanode in the pipeline. DFSOutputStream also maintains an internal queue of packets that are waiting to be acknowledged by datanodes, called the ack queue. A packet is removed from the ack queue only when it has been acknowledged by all the datanodes in the pipeline. 
- If a datanode fails while data is being written to it, then the following actions are taken, which are transparent to the client writing the data. First the pipeline is closed, and any packets in the ack queue are added to the front of the data queue so that datanodes that are downstream from the failed node will not miss any packets.
- The current block on the good datanodes is given a new identity, which is communicated to the namenode, so that the partial block on the failed datanode will be deleted if the failed datanode recovers later on. The failed datanode is removed from the pipeline and the remainder of the block’s data is written to the two good datanodes in the pipeline.
- The namenode notices that the block is under-replicated, and it arranges for a further replica to be created on another node. Subsequent blocks are then treated as normal. It’s possible, but unlikely, that multiple datanodes fail while a block is being written. - As long as dfs.replication.min replicas (default one) are written, the write will succeed, and the block will be asynchronously replicated across the cluster until its target replication factor is reached (dfs.replication, which defaults to three). 
- When the client has finished writing data, it calls close() on the stream. This action flushes all the remaining packets to the datanode pipeline and waits for acknowledgments before contacting the namenode to signal that the file is complete.
- The namenode already knows which blocks the file is made up of (via Data Streamer asking for block allocations), so it only has to wait for blocks to be minimally replicated before returning successfully.

3)
#HDFS HADLES FAILURES DURING FILE WRITE IN FOLLOWING WAY:

- While writing the blocks in the data nodes one of the writing to the data node fails. Immediately the pipeline will be closed and the data nodes which are working fine are given a new identity. 
- This change is conveyed to Name node. Once this is conveyed the failed data node is having partial data of the block that data is removed because it may cause confusion if the data node gets repaired and come back live.
- The remaining blocks in the queue are moved to the data queue and they are written into the two data nodes(considered replication factor as 3)once done the hdfs recognizes that block has been under replicated and hence generates another cop of the block in some fully functional data node.
