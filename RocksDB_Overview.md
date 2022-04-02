网址 https://github.com/facebook/rocksdb/wiki/RocksDB-Overview

有道

# RocksDB Overview

https://github.com/facebook/rocksdb.wiki.git
## 1. Introduction
RocksDB started at Facebook as a storage engine for server workloads on various storage media, with the initial focus on fast storage (especially Flash storage). It is a C++ library to store keys and values, which are arbitrarily-sized byte streams. It supports both point lookups and range scans, and provides different types of ACID guarantees.  
RocksDB最初在Facebook时是一个存储引擎，用于在各种存储媒体上处理服务器工作负载，最初专注于快速存储(尤其是Flash存储)。 它是一个c++标准库，用于存储键和值，它们是任意大小的字节流。 它支持点查找和范围扫描，并提供不同类型的ACID保证。  

A balance is struck between customizability and self-adaptability. RocksDB features highly flexible configuration settings that may be tuned to run on a variety of production environments, including SSDs, hard disks, ramfs, or remote storage. It supports various compression algorithms and good tools for production support and debugging. On the other hand, efforts are also made to limit the number of knobs, to provide good enough out-of-box performance, and to use some adaptive algorithms wherever applicable.  
在可定制性和自适应性之间取得了平衡。 RocksDB具有高度灵活的配置设置，可以调优以运行在各种生产环境中，包括ssd、硬盘、ramfs或远程存储。 它支持各种压缩算法和用于生产支持和调试的好工具。 另一方面，还努力限制旋钮的数量，以提供足够好的开箱即用性能，并在适用的地方使用一些自适应算法。 

RocksDB borrows significant code from the open source leveldb project as well as ideas from Apache HBase. The initial code was forked from open source leveldb 1.5. It also builds upon code and ideas that were developed at Facebook before RocksDB.  
RocksDB借鉴了开源leveldb项目的大量代码以及Apache HBase的想法。 最初的代码是从开源的leveldb 1.5派生出来的。 它也建立在Facebook在RocksDB之前开发的代码和想法之上。  

## 2. Assumptions and Goals
#### Performance:
The primary design point for RocksDB is that it should be performant for fast storage and for server workloads. It should support efficient point lookups as well as range scans. It should be configurable to support high random-read workloads, high update workloads or a combination of both. Its architecture should support easy tuning of trade-offs for different workloads and hardware.  
RocksDB的主要设计要点是，它应该为快速存储和服务器工作负载提供性能。它应该支持高效的点查找和范围扫描。它应该是可配置的，以支持高随机读工作负载、高更新工作负载或两者的组合。它的架构应该支持针对不同的工作负载和硬件进行轻松的权衡调整。

#### Production Support:
RocksDB should be designed in such a way that it has built-in support for tools and utilities that help deployment and debugging in production environments. If the storage engine cannot yet be able to automatically adapt the application and hardware, we will provide some parameters to allow users to tune performance.  
RocksDB的设计应该内置对工具和实用程序的支持，以帮助在生产环境中进行部署和调试。如果存储引擎还不能自动适应应用程序和硬件，我们将提供一些参数来允许用户调优性能。

#### Compatibility:
Newer versions of this software should be backward compatible, so that existing applications do not need to change when upgrading to newer releases of RocksDB. Unless using newly provided features, existing applications also should be able to revert to a recent old release. See RocksDB Compatibility Between Different Releases.  
该软件的新版本应该是向后兼容的，这样在升级到RocksDB的新版本时，现有的应用程序不需要更改。除非使用新提供的特性，否则现有的应用程序也应该能够恢复到最近的旧版本。参见RocksDB不同版本之间的兼容性。

## 3. High Level Architecture
RocksDB is a storage engine library of key-value store interface where keys and values are arbitrary byte streams. RocksDB organizes all data in sorted order and the common operations are Get(key), NewIterator(), Put(key, val), Delete(key), and SingleDelete(key).  
RocksDB是一个键-值存储接口的存储引擎库，其中键和值是任意的字节流。RocksDB按照排序的顺序组织所有数据，常用的操作有Get(key)、NewIterator()、Put(key, val)、Delete(key)、SingleDelete(key)。

The three basic constructs of RocksDB are memtable, sstfile and logfile. The memtable is an in-memory data structure - new writes are inserted into the memtable and are optionally written to the logfile (aka. Write Ahead Log(WAL)). The logfile is a sequentially-written file on storage. When the memtable fills up, it is flushed to a sstfile on storage and the corresponding logfile can be safely deleted. The data in an sstfile is sorted to facilitate easy lookup of keys.  
RocksDB的三个基本结构是memtable, sstfile和logfile。memtable是一个内存中的数据结构——新的写操作被插入到memtable中，并且可以选择写入日志文件。写日志(细胞膜))。日志文件是一个顺序写入存储的文件。当memtable被填满时，它被刷新到存储上的sstfile，相应的日志文件可以被安全地删除。sstfile中的数据进行了排序，以便于方便地查找键。

The default format of sstfile is described in more details here.  
这里详细描述了sstfile的默认格式。
![image](https://user-images.githubusercontent.com/62277872/119747261-310fb300-be47-11eb-92c3-c11719fa8a0c.png)

## 4. Features
#### Column Families
RocksDB supports partitioning a database instance into multiple column families. All databases are created with a column family named "default", which is used for operations where column family is unspecified.  
RocksDB支持将数据库实例划分为多个列族。所有数据库都使用名为“default”的列族创建，该列族用于未指定列族的操作。

RocksDB guarantees users a consistent view across column families, including after crash recovery when WAL is enabled or atomic flush is enabled. It also supports atomic cross-column family operations via the WriteBatch API.  
RocksDB保证用户跨列族获得一致的视图，包括启用WAL或启用原子刷新时的崩溃恢复。它还通过WriteBatch API支持原子跨列族操作。

#### Updates
A Put API inserts a single key-value to the database. If the key already exists in the database, the previous value will be overwritten. A Write API allows multiple keys-values to be atomically inserted, updated, or deleted in the database. The database guarantees that either all of the keys-values in a single Write call will be inserted into the database or none of them will be inserted into the database. If any of those keys already exist in the database, previous values will be overwritten. DeleteRange API can be used to delete all keys from a range.  
Put API向数据库插入一个键值。如果该键已经存在于数据库中，那么以前的值将被覆盖。Write API允许在数据库中原子地插入、更新或删除多个键值。数据库保证将单个Write调用中的所有键值插入到数据库中，或者不将任何键值插入到数据库中。如果这些键中的任何一个已经存在于数据库中，那么以前的值将被覆盖。DeleteRange API可用于删除一个范围内的所有键。

#### Gets, Iterators and Snapshots
Keys and values are treated as pure byte streams. There is no limit to the size of a key or a value. The Get API allows an application to fetch a single key-value from the database. The MultiGet API allows an application to retrieve a bunch of keys from the database. All the keys-values returned via a MultiGet call are consistent with one-another.  
键和值被视为纯字节流。键或值的大小没有限制。Get API允许应用程序从数据库中获取单个键值。MultiGet API允许应用程序从数据库中检索一串键。通过MultiGet调用返回的所有键值都是一致的。

All data in the database is logically arranged in sorted order. An application can specify a key comparison method that specifies a total ordering of keys. An Iterator API allows an application to do a range scan on the database. The Iterator can seek to a specified key and then the application can start scanning one key at a time from that point. The Iterator API can also be used to do a reverse iteration of the keys in the database. A consistent-point-in-time view of the database is created when the Iterator is created. Thus, all keys returned via the Iterator are from a consistent view of the database.  
数据库中的所有数据在逻辑上按顺序排列。应用程序可以指定一个键比较方法，该方法指定键的总顺序。Iterator API允许应用程序对数据库进行范围扫描。Iterator可以查找指定的键，然后应用程序可以从这个点开始一次扫描一个键。还可以使用Iterator API对数据库中的键进行反向迭代。在创建迭代器时，将创建数据库的一致时间点视图。因此，通过Iterator返回的所有键都来自数据库的一致视图。

A Snapshot API allows an application to create a point-in-time view of a database. The Get and Iterator APIs can be used to read data from a specified snapshot. In a sense, a Snapshot and an Iterator both provide a point-in-time view of the database, but their implementations are different. Short-lived/foreground scans are best done via an iterator while long-running/background scans are better done via a snapshot. An Iterator keeps a reference count on all underlying files that correspond to that point-in-time-view of the database - these files are not deleted until the Iterator is released. A Snapshot, on the other hand, does not prevent file deletions; instead the compaction process understands the existence of Snapshots and promises never to delete a key that is visible in any existing Snapshot.  
Snapshot API允许应用程序创建一个数据库的时间点视图。Get和Iterator api可用于从指定的快照读取数据。从某种意义上说，Snapshot和Iterator都提供了数据库的时间点视图，但它们的实现是不同的。短期扫描/前台扫描最好通过迭代器完成，而长期扫描/后台扫描最好通过快照完成。迭代器会对与该数据库的时间点视图对应的所有底层文件保持引用计数——这些文件在迭代器释放之前不会被删除。另一方面，快照不阻止文件删除;相反，压缩过程理解快照的存在，并承诺永远不会删除任何现有快照中可见的键。

Snapshots are not persisted across database restarts: a reload of the RocksDB library (via a server restart) releases all pre-existing Snapshots.  
快照在数据库重启时不会被持久化:重新加载RocksDB库(通过服务器重启)会释放所有现有的快照。

#### Transactions
RocksDB supports multi-operational transactions. It supports both of optimistic and pessimistic mode. See Transactions.  
RocksDB支持多操作事务。它支持乐观模式和悲观模式。看到交易。

#### Prefix Iterators
Most LSM-tree engines cannot support an efficient range scan API because it needs to look into multiple data files. But, most applications do not do pure-random scans of key ranges in the database; instead, applications typically scan within a key-prefix. RocksDB uses this to its advantage. Applications can configure a Options.prefix_extractor to enable a key-prefix based filtering. When Options.prefix_extractor is set, a hash of the prefix is also added to the Bloom. An Iterator that specifies a key-prefix (in ReadOptions) will use the Bloom Filter to avoid looking into data files that do not contain keys with the specified key-prefix. See Prefix-Seek.  
大多数LSM-tree引擎不能支持有效的范围扫描API，因为它需要查看多个数据文件。但是，大多数应用程序不会对数据库中的键范围进行纯随机扫描;相反，应用程序通常在一个键-前缀内进行扫描。RocksDB利用了这一点。应用程序可以配置一个选项。Prefix_extractor来启用基于键-前缀的过滤。当选择。设置prefix_extractor后，还将该前缀的哈希值添加到Bloom中。指定键前缀的迭代器(在ReadOptions中)将使用Bloom Filter来避免查找不包含指定键前缀的键的数据文件。看到Prefix-Seek。

#### Persistence
RocksDB has a Write Ahead Log (WAL). All write operations (Put, Delete and Merge) are stored in an in-memory buffer called the memtable as well as optionally inserted into WAL. On restart, it re-processes all the transactions that were recorded in the log.  
RocksDB有一个预写日志(Write Ahead Log, WAL)。所有写操作(Put, Delete和Merge)都存储在内存缓冲区中，称为memtable，也可以插入到WAL中。在重新启动时，它会重新处理日志中记录的所有事务。

WAL can be configured to be stored in a directory different from the directory where the SST files are stored. This is necessary for those cases in which you might want to store all data files in non-persistent fast storage. At the same time, you can ensure no data loss by putting all transaction logs on slower but persistent storage.  
可以将WAL配置为存储在与SST文件存储目录不同的目录中。对于希望将所有数据文件存储在非持久快速存储中的情况，这是必要的。同时，您可以通过将所有事务日志放在较慢但持久的存储中来确保没有数据丢失。

Each Put has a flag, set via WriteOptions, which specifies whether or not the Put should be inserted into the transaction log. The WriteOptions may also specify whether or not a fsync call is issued to the transaction log before a Put is declared to be committed.  
**每个Put都有一个通过WriteOptions设置的标志，该标志指定是否应该将Put插入事务日志中。WriteOptions还可以指定在声明Put被提交之前是否向事务日志发出fsync调用。**

Internally, RocksDB uses a batch-commit mechanism to batch transactions into the log so that it can potentially commit multiple transactions using a single fsync call.  
**在内部，RocksDB使用批提交机制将事务批处理到日志中，这样它就可以使用一个fsync调用提交多个事务。**

#### Data Checksuming
RocksDB uses a checksum to detect corruptions in storage. These checksums are for each SST file block (typically between 4K to 128K in size). A block, once written to storage, is never modified. RocksDB also maintains a Full File Checksum.  
RocksDB使用校验和来检测存储中的损坏。这些校验和针对每个SST文件块(大小通常在4K到128K之间)。块一旦写入存储器，就永远不会被修改。RocksDB还维护一个完整文件校验和。

RocksDB dynamically detects and utilizes CPU checksum offload support.  
RocksDB动态检测并利用CPU校验和卸载支持。

#### Multi-Threaded Compactions
In the presence of ongoing writes, compactions are needed for space efficiency, read (query) efficiency, and timely data deletion. Compaction removes key-value bindings that have been deleted or overwritten, and re-organizes data for query efficiency. Compactions may occur in multiple threads if configured.  
在进行写操作的情况下，为了提高空间效率、读取(查询)效率和及时删除数据，需要进行压缩。压缩删除已删除或覆盖的键值绑定，并重新组织数据以提高查询效率。如果配置，压缩可能在多个线程中发生。

The entire database is stored in a set of sstfiles. When a memtable is full, its content is written out to a file in Level-0 (L0) of the LSM tree. RocksDB removes duplicate and overwritten keys in the memtable when it is flushed to a file in L0. In compaction, some files are periodically read in and merged to form larger files, often going into the next LSM level (such as L1, up to Lmax).  
整个数据库存储在一组sstfile中。当一个memtable被填满时，它的内容被写到LSM树的0级(L0)文件中。当将memtable刷新到L0中的一个文件时，RocksDB删除了memtable中重复和覆盖的键。在压缩过程中，定期读入一些文件并合并成更大的文件，通常进入下一个LSM级别(例如L1到Lmax)。

The overall write throughput of an LSM database directly depends on the speed at which compactions can occur, especially when the data is stored in fast storage like SSD or RAM. RocksDB may be configured to issue concurrent compaction requests from multiple threads. It is observed that sustained write rates may increase by as much as a factor of 10 with multi-threaded compaction when the database is on SSDs, as compared to single-threaded compactions.  
LSM数据库的整体写吞吐量直接取决于压缩发生的速度，特别是当数据存储在诸如SSD或RAM之类的快速存储中时。RocksDB可以配置为从多个线程发出并发压缩请求。可以观察到，与单线程紧凑相比，在数据库位于ssd上时，使用多线程紧凑可以增加10倍的持续写入速率。

#### Compaction Styles
Both Level Style Compaction and Universal Style Compaction store data in a fixed number of logical levels in the database. More recent data is stored in Level-0 (L0) and older data in higher-numbered levels, up to Lmax. Files in L0 may have overlapping keys, but files in other levels generally form a single sorted run per level.  
Level Style Compaction和Universal Style Compaction都将数据存储在数据库中固定数量的逻辑级别中。最近的数据存储在Level-0 (L0)中，旧的数据存储在更高编号的级别中，一直到Lmax。L0中的文件可能有重叠的键，但其他级别的文件通常会在每个级别上单独有序运行。

Level Style Compaction (default) typically optimizes disk footprint vs. logical database size (space amplification) by minimizing the files involved in each compaction step: merging one file in Ln with all its overlapping files in Ln+1 and replacing them with new files in Ln+1.  
级别样式压缩(默认)通常通过最小化每个压缩步骤中涉及的文件来优化磁盘占用空间和逻辑数据库大小(空间放大):将Ln中的一个文件与Ln+1中的所有重叠文件合并，并用Ln+1中的新文件替换它们。

Universal Style Compaction typically optimizes total bytes written to disk vs. logical database size (write amplification) by merging potentially many files and levels at once, requiring more temporary space. Universal typically results in lower write-amplification but higher space- and read-amplification than Level Style Compaction.  
Universal Style Compaction通常通过一次合并多个文件和级别来优化写入磁盘的字节总数和逻辑数据库大小(写入放大)，这需要更多的临时空间。与级别样式压缩相比，通用压缩通常会导致更低的写放大，但更高的空间和读放大。

FIFO Style Compaction drops oldest file when obsolete and can be used for cache-like data. In FIFO compaction, all files are in level 0. When total size of the data exceeds configured size (CompactionOptionsFIFO::max_table_files_size), we delete the oldest table file.  
FIFO风格压缩在过时时会删除最旧的文件，并可用于缓存类数据。在FIFO压缩中，所有文件都处于0级。当数据的总大小超过配置的大小(CompactionOptionsFIFO::max_table_files_size)时，我们删除最老的表文件。

We also enable developers to develop and experiment with custom compaction policies. For this reason, RocksDB has appropriate hooks to switch off the inbuilt compaction algorithm and has other APIs to allow applications to operate their own compaction algorithms. Options.disable_auto_compaction, if set, disables the native compaction algorithm. The GetLiveFilesMetaData API allows an external component to look at every data file in the database and decide which data files to merge and compact. Call CompactFiles to compact files you want. The DeleteFile API allows applications to delete data files that are deemed obsolete.  
我们还允许开发人员开发和试验自定义压缩策略。因此，RocksDB有适当的钩子来关闭内置的压缩算法，并有其他api允许应用程序操作自己的压缩算法。选项。如果设置了Disable_auto_compaction，则禁用本机压缩算法。GetLiveFilesMetaData API允许外部组件查看数据库中的每个数据文件，并决定合并和压缩哪些数据文件。调用CompactFiles来压缩你想要的文件。DeleteFile API允许应用程序删除被认为过时的数据文件。

#### Metadata storage
A manifest log file is used to record all the database state changes. The compaction process adds new files and deletes existing files from the database, and it makes these operations persistent by recording them in the MANIFEST.  
清单日志文件用于记录所有数据库状态的更改。压缩过程从数据库中添加新文件并删除现有文件，并且通过在MANIFEST中记录这些操作，使这些操作具有持久性。

#### Avoiding Stalls
Background compaction threads are also used to flush memtable contents to a file on storage. If all background compaction threads are busy doing long-running compactions, then a sudden burst of writes can fill up the memtable(s) quickly, thus stalling new writes. This situation can be avoided by configuring RocksDB to keep a small set of threads explicitly reserved for the sole purpose of flushing memtable to storage.  
后台压缩线程也用于将memtable内容刷新到存储器上的一个文件中。如果所有的后台压缩线程都在忙着执行长时间运行的压缩，那么突发的写操作会很快地填满memtable，从而导致新的写入延迟。这种情况可以通过配置RocksDB来避免，它会为将memtable刷新到存储的唯一目的而显式地保留一小组线程。

#### Compaction Filter
Some applications may want to process keys at compaction time. For example, a database with inherent support for time-to-live (TTL) may remove expired keys. This can be done via an application-defined Compaction-Filter. If the application wants to continuously delete data older than a specific time, it can use the compaction filter to drop records that have expired. The RocksDB Compaction Filter gives control to the application to modify the value of a key or to drop a key entirely as part of the compaction process. For example, an application can continuously run a data sanitizer as part of the compaction.  
一些应用程序可能希望在压缩时处理键。例如，固有支持生存时间(TTL)的数据库可以删除过期的密钥。这可以通过应用程序定义的压缩过滤器来实现。如果应用程序希望不断删除比特定时间更早的数据，它可以使用压缩过滤器删除已经过期的记录。RocksDB Compaction Filter让应用程序可以修改一个键的值，或者作为压缩过程的一部分删除一个键。例如，应用程序可以在压缩过程中持续运行数据清理程序。

#### ReadOnly Mode
A database may be opened in ReadOnly mode, in which the database guarantees that the application may not modify anything in the database. This results in much higher read performance because oft-traversed code paths avoid locks completely.  
数据库可以以ReadOnly模式打开，在这种模式下，数据库保证应用程序不会修改数据库中的任何内容。这将导致更高的读取性能，因为经常遍历的代码路径完全避免了锁。

#### Database Debug Logs
By default, RocksDB writes detailed logs to a file named LOG*. These are mostly used for debugging and analyzing a running system. Users can choose different log levels. This LOG may be configured to roll at a specified periodicity. The logging interface is pluggable. Users can plug in a different logger. See Logger.

#### Data Compression
RocksDB supports lz4, zstd, snappy, zlib, and lz4_hc compression, as well as xpress under Windows. RocksDB may be configured to support different compression algorithms for data at the bottommost level, where 90% of data lives. A typical installation might configure ZSTD (or Zlib if not available) for the bottom-most level and LZ4 (or Snappy if it is not available) for other levels. See Compression.  
默认情况下，RocksDB会将详细日志写到LOG*文件中。它们主要用于调试和分析运行中的系统。用户可以选择不同的日志级别。可以将此LOG配置为按指定的周期滚动。日志接口是可插拔的。用户可以插入不同的日志记录器。看到日志记录器。

#### Full Backups and Replication
RocksDB provides a backup API, BackupEngine. You can read more about it here: How to backup RocksDB  
RocksDB提供备份API BackupEngine。你可以在这里阅读更多:如何备份RocksDB

RocksDB itself is not a replicated, but it provides some helper functions to enable users to implement their replication system on top of RocksDB, see Replication Helpers.  
RocksDB本身不是一个复制的，但它提供了一些帮助函数，使用户可以在RocksDB的基础上实现他们的复制系统，参见replication Helpers。

#### Support for Multiple Embedded Databases in the same process
A common use-case for RocksDB is that applications inherently partition their data set into logical partitions or shards. This technique benefits application load balancing and fast recovery from faults. This means that a single server process should be able to operate multiple RocksDB databases simultaneously. This is done via an environment object named Env. Among other things, a thread pool is associated with an Env. If applications want to share a common thread pool (for background compactions) among multiple database instances, then it should use the same Env object for opening those databases.  
RocksDB的一个常见用例是，应用程序固有地将其数据集划分为逻辑分区或分片。这种技术有利于应用程序负载平衡和快速从故障中恢复。这意味着一个服务器进程应该能够同时操作多个RocksDB数据库。这是通过一个名为Env的环境对象来完成的。其中，线程池与Env相关联。如果应用程序希望在多个数据库实例之间共享一个公共线程池(用于后台压缩)，那么它应该使用相同的Env对象来打开这些数据库。

Similarly, multiple database instances may share the same block cache or rate limiter.  
类似地，多个数据库实例可能共享相同的块缓存或速率限制器。

#### Block Cache -- Compressed and Uncompressed Data
RocksDB uses a LRU cache for blocks to serve reads. The block cache is partitioned into two individual caches: the first caches uncompressed blocks and the second caches compressed blocks in RAM. If a compressed block cache is configured, users may wish to enable direct I/O to prevent redundant caching of the same data in OS page cache.  
RocksDB使用LRU缓存块来提供读操作。块缓存被划分为两个独立的缓存:第一个缓存未压缩的块，第二个缓存压缩的块。如果配置了压缩块缓存，用户可能希望启用直接I/O，以防止在操作系统页面缓存中重复缓存相同的数据。

#### Table Cache
The Table Cache is a construct that caches open file descriptors. These file descriptors are for sstfiles. An application can specify the maximum size of the Table Cache, or configure RocksDB to always keep all files open, to achieve better performance.  
Table Cache是一种用来缓存打开文件描述符的结构。这些文件描述符是用于sstfiles的。应用程序可以指定Table Cache的最大大小，或者配置RocksDB始终保持所有文件打开，以获得更好的性能。

#### I/O Control
RocksDB allows users to configure I/O from and to SST files in different ways. Users can enable direct I/O so that RocksDB takes full control to the I/O and caching. An alternative is to leverage some options to allow users to hint about how I/O should be executed. They can suggest RocksDB to call fadvise in files to read, call periodic range sync in files being appended, enable direct I/O, etc... See IO for more details.  
RocksDB允许用户以不同的方式配置来自和到SST文件的I/O。用户可以启用直接I/O，这样RocksDB就可以完全控制I/O和缓存。另一种方法是利用一些选项允许用户提示应该如何执行I/O。他们可以建议RocksDB在文件中调用fadvise来读取，在被追加的文件中调用周期性范围同步，启用直接I/O等等……更多细节请参见IO。

#### Stackable DB
RocksDB has a built-in wrapper mechanism to add functionality as a layer above the code database kernel. This functionality is encapsulated by the StackableDB API. For example, the time-to-live functionality is implemented by a StackableDB and is not part of the core RocksDB API. This approach keeps the code modularized and clean.  
RocksDB有一个内置的包装器机制，可以在代码数据库内核之上添加一层功能。这个功能由StackableDB API封装。例如，time-to-live功能是由StackableDB实现的，而不是核心RocksDB API的一部分。这种方法使代码保持模块化和干净。（待分析）

#### Memtables:
- Pluggable Memtables:  
The default implementation of the memtable for RocksDB is a skiplist. The skiplist is a sorted set, which is a necessary construct when the workload interleaves writes with range-scans. Some applications do not interleave writes and scans, however, and some applications do not do range-scans at all. For these applications, a sorted set may not provide optimal performance. For this reason, RocksDB's memtable is pluggable. Some alternative implementations are provided. Three memtables are part of the library: a skiplist memtable, a vector memtable and a prefix-hash memtable. A vector memtable is appropriate for bulk-loading data into the database. Every write inserts a new element at the end of the vector; when it is time to flush the memtable to storage the elements in the vector are sorted and written out to a file in L0. A prefix-hash memtable allows efficient processing of gets, puts and scans-within-a-key-prefix. Although the pluggability of memtable is not provided as a public API, it is possible for an application to provide its own implementation of a memtable, in a private fork.  
RocksDB的memtable的默认实现是skiplist。skiplist是一个排序的集合，当工作负载与范围扫描交叉写入时，这是一个必要的构造。然而，有些应用程序并不交叉写入和扫描，有些应用程序根本不进行范围扫描。对于这些应用程序，排序集可能不能提供最佳性能。因此，RocksDB的memtable是可插拔的。还提供了一些替代实现。库中有三个memtable: skiplist memtable、vector memtable和prefix-hash memtable。vector memtable适用于将数据批量加载到数据库中。每次写操作都在vector的末尾插入一个新元素;当需要刷新memtable以存储vector中的元素时，将对其进行排序并写入L0中的文件。前缀-哈希memtable允许高效地处理key-prefix中的get、puts和scan。虽然memtable的可插拔性不是作为公共API提供的，但应用程序可以在私有分支中提供自己的memtable实现。

- Memtable Pipelining  
RocksDB supports configuring an arbitrary number of memtables for a database. When a memtable is full, it becomes an immutable memtable and a background thread starts flushing its contents to storage. Meanwhile, new writes continue to accumulate to a newly allocated memtable. If the newly allocated memtable is filled up to its limit, it is also converted to an immutable memtable and is inserted into the flush pipeline. The background thread continues to flush all the pipelined immutable memtables to storage. This pipelining increases write throughput of RocksDB, especially when it is operating on slow storage devices.  
RocksDB支持为数据库配置任意数量的memtables。当一个memtable被填满时，它就变成一个不可变的memtable，一个后台线程开始将它的内容刷新到存储器中。同时，新写操作继续累积到一个新分配的memtable中。如果新分配的memtable被填满，它也会被转换为一个不可变的memtable，并被插入到刷新管道中。后台线程继续将管道中所有不可变的memtable刷新到存储中。这种流水线增加了RocksDB的写吞吐量，特别是当它在低速存储设备上运行时。

-  Garbage Collection during Memtable Flush:  
When a memtable is being flushed to storage, an inline-compaction process is executed. Garbages are removed in the same way as compactions. Duplicate updates for the same key are removed from the output stream. Similarly, if an earlier put is hidden by a later delete, then the put is not written to the output file at all. This feature reduces the size of data on storage and write amplification greatly, for some workloads.  
当一个memtable被刷新到存储器时，会执行一个内联压缩过程。垃圾清除的方法与压实的方法相同。从输出流中删除相同键的重复更新。类似地，如果前面的放置被后面的删除所隐藏，则该放置根本不会写入输出文件。对于某些工作负载，这个特性减少了存储上的数据大小并大大增加了写。

#### Merge Operator
RocksDB natively supports three types of records, a Put record, a Delete record and a Merge record. When a compaction process encounters a Merge record, it invokes an application-specified method called the Merge Operator. The Merge can combine multiple Put and Merge records into a single one. This powerful feature allows applications that typically do read-modify-writes to avoid the reads altogether. It allows an application to record the intent-of-the-operation as a Merge Record, and the RocksDB compaction process lazily applies that intent to the original value. This feature is described in detail in Merge Operator  
RocksDB本身支持三种类型的记录，Put记录、Delete记录和Merge记录。当压缩流程遇到Merge记录时，它会调用一个应用程序指定的方法，称为Merge Operator。Merge可以将多个Put和Merge记录合并为一个记录。这个强大的特性允许通常执行读-修改-写操作的应用程序完全避免读操作。它允许应用程序将操作意图记录为Merge record，而RocksDB压缩过程将该意图惰性地应用到原始值。该特性在合并操作符中有详细描述

#### DB ID
A globally unique ID created at the time of database creation and stored in IDENTITY file in the DB folder by default. Optionally it can only be stored in the MANIFEST file. Storing in the MANIFEST file is recommended.  
在创建数据库时创建的全局唯一ID，默认情况下存储在DB文件夹中的IDENTITY文件中。或者，它只能存储在MANIFEST文件中。建议存储在MANIFEST文件中。

## 5. Tools
There are a number of interesting tools that are used to support a database in production. The sst_dump utility dumps all the keys-values in a sst file, as well as other information. The ldb tool can put, get, scan the contents of a database. ldb can also dump contents of the MANIFEST, it can also be used to change the number of configured levels of the database. See Administration and Data Access Tool for details.  
有许多有趣的工具用于在生产环境中支持数据库。sst_dump实用程序转储一个sst文件中的所有键值以及其他信息。ldb工具可以放置、获取、扫描数据库的内容。ldb还可以转储MANIFEST的内容，它还可以用来更改配置的数据库级别的数量。详情请参见管理和数据访问工具。

## 6. Tests
There are a bunch of unit tests that test specific features of the database. A make check command runs all unit tests. The unit tests trigger specific features of RocksDB and are not designed to test data correctness at scale. The db_stress test is used to validate data correctness at scale. See Stress-test.  
有一堆测试数据库特定特性的单元测试。生成检查命令运行所有单元测试。单元测试触发了RocksDB的特定特性，并不是为了大规模测试数据正确性而设计的。db_stress测试用于大规模验证数据的正确性。看到压力测试。

## 7. Performance
RocksDB performance is benchmarked via a utility called db_bench. db_bench is part of the RocksDB source code. Performance results of a few typical workloads using Flash storage are described here. You can also find RocksDB performance results for in-memory workload here.  
RocksDB的性能是通过一个名为db_bench的实用程序进行基准测试的。db_bench是RocksDB源代码的一部分。这里描述了使用Flash存储的几个典型工作负载的性能结果。您还可以在这里找到针对内存工作负载的RocksDB性能结果。
