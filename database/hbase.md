* HBase 是一个分布式、可扩展、面向列的 NoSQL 数据库，
主要是LSM Tree的思想。  
* 它利用 HDFS 作为文件存储系统，利用 MapReduce
进行海量数据处理，利用 ZooKeeper 作为协同服务。  
* 主要是为了解决海量数据存储和实时随机读写的问题。  

物理磁盘上的文件，HDFS 上存储的是 HFile 文件，HFile 文件中存储的是
KeyValue 数据。  
key 是 rowkey + column family + column qualifier，value 是
byte[] 数组。  

HBase 的写入流程：
1. 客户端向 HMaster 请求写入数据，HMaster 将数据分配给一个 RegionServer。
2. RegionServer 将数据写入 HLog（预写日志），然后将数据写入 MemStore（内存中）。
3. 当 MemStore 的大小达到一定阈值时，RegionServer 将 MemStore 中的数据刷写
到磁盘上的 HFile 文件中。  
4. 当 HFile 文件的数量达到一定阈值时，RegionServer 会将 HFile 文件合并成
一个 HFile 文件。  

HBase 的读取流程：
1. 客户端向 HMaster 请求读取数据，HMaster 将数据分配给一个 RegionServer。
2. RegionServer 从 MemStore 中读取数据，如果 MemStore 中没有数据，
则从 HFile 文件中读取数据。  
3. RegionServer 将数据返回给客户端。  
4. 如果客户端需要读取的数据不在一个 RegionServer 上，则 HMaster 会将数据
分配给多个 RegionServer。  



