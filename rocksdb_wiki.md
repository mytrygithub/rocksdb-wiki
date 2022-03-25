原文网址：https://github.com/facebook/rocksdb/wiki

（有道+自己翻译）


# Welcome to RocksDB
RocksDB is a storage engine with key/value interface, where keys and values are arbitrary byte streams. It is a C++ library. It was developed at Facebook based on LevelDB and provides backwards-compatible support for LevelDB APIs.  
RocksDB是一个k/v存储引擎，key和value支持二进制字节流。是一个c++库。Facebook基于levelDB开发的兼容levelDB的库。

RocksDB supports various storage hardware, with fast flash as the initial focus. It uses a Log Structured Database Engine for storage, is written entirely in C++, and has a Java wrapper called RocksJava. See [[RocksJava Basics]].  
RocksDB支持各种存储硬件，主要支持快速存储flash。它是一个Log Structured Database Engine，c++实现，有java接口封装。

RocksDB can adapt to a variety of production environments, including pure memory, Flash, hard disks or remote storage. Where RocksDB cannot automatically adapt, highly flexible configuration settings are provided to allow users to tune it for them. It supports various compression algorithms and good tools for production support and debugging.  
Rocksdb可以适配各种产品环境，包括纯内存、flash、磁盘和远程存储。当自适应无法满足条件时，高度灵活的配置可以用于用户自己调整适配。支持多种压缩算法和工具和调试。

## Features
* Designed for application servers wanting to store up to a few terabytes of data on local or remote storage systems.  
设计用于存储几TB的数据在本地或者远程存储系统。
* Optimized for storing small to medium size key-values on fast storage -- flash devices or in-memory  
优化存储小的或中等大小的k/v在快速存储-- flash或者内存。
* It works well on processors with many cores  
多核环境运行良好

## Features Not in LevelDB
RocksDB introduces dozens of new major features. See [the list of features not in LevelDB](https://github.com/facebook/rocksdb/wiki/Features-Not-in-LevelDB).  
Rocksdb有几十中主要特性。参见[the list of features not in LevelDB](https://github.com/facebook/rocksdb/wiki/Features-Not-in-LevelDB).

## Getting Started
For a complete Table of Contents, see the sidebar to the right. Most readers will want to start with the [Overview](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview) and the [Basic Operations](https://github.com/facebook/rocksdb/wiki/Basic-Operations) section of the Developer's Guide. Get your initial options set-up following [[Setup Options and Basic Tuning]]. Also check [[RocksDB FAQ]]. There is also a [[RocksDB Tuning Guide]] for advanced RocksDB users.  
完整目录参加右侧列表。大部分读者从[Overview](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview)开始，或者开发者从 [Basic Operations](https://github.com/facebook/rocksdb/wiki/Basic-Operations)开始。初始配置参考右侧"Setup Options and Basic Tuning",或者参考"RocksDB FAQ"。高级开发这可以参考"RocksDB Tuning Guide".

Check [INSTALL.md](https://github.com/facebook/rocksdb/blob/main/INSTALL.md) for instructions on how to build Rocksdb.  
INSTALL.md介绍构建过程。

## Releases
RocksDB releases are done in github. For Java users, Maven Artifacts are also updated. See [[RocksDB Release Methodology]].  
Rocksdb的release在github完成。对于java用户，Maven Artifacts也会更新。参见"RocksDB Release Methodology".

## Contributing to RocksDB
You are welcome to send pull requests to contribute to RocksDB code base! Check [[RocksDB-Contribution-Guide]] for guideline.  
欢迎发送 pull requests提交代码，参考流程"RocksDB-Contribution-Guide".

## Troubleshooting and asking for help
Follow [[RocksDB Troubleshooting Guide]] for the guidelines.  
"RocksDB Troubleshooting Guide"用于定位问题。

## Blog 
* Check out our blog at [rocksdb.org/blog](http://rocksdb.org/blog)  
blog 参见[rocksdb.org/blog](http://rocksdb.org/blog)

## Project History
* [The History of RocksDB](http://rocksdb.blogspot.com/2013/11/the-history-of-rocksdb.html)
* [Under the Hood: Building and open-sourcing RocksDB](https://www.facebook.com/notes/facebook-engineering/under-the-hood-building-and-open-sourcing-rocksdb/10151822347683920).

## Links 
* [Examples](https://github.com/facebook/rocksdb/tree/main/examples)
* [Official Blog](http://rocksdb.org/blog/)
* [Stack Overflow: RocksDB](https://stackoverflow.com/questions/tagged/rocksdb)
* [Talks](https://github.com/facebook/rocksdb/wiki/Talks)

## Contact
* [RocksDB Google Group](https://groups.google.com/forum/#!forum/rocksdb)
* [RocksDB Facebook Group](https://www.facebook.com/groups/rocksdb.dev/)
* [RocksDB Github Issues](https://github.com/facebook/rocksdb/issues)
* [Asking For Help](https://github.com/facebook/rocksdb/wiki/RocksDB-Troubleshooting-Guide#asking-for-help) <br>
We use github issues only for bug reports, and use RocksDB's Google Group or Facebook Group for other issues. It’s not always clear to users whether it is RocksDB bug or not. Pick one using your best judgement.  
github issues 只用于提交bug，Google Group 或者 Facebook Group黄永玉其他内容。如果你不清晰是否是bug，选择你认为正确的就好。



