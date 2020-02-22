# Buffer pool manager
## 1. General database concepts
### 1.1 database vs database management system
* Database is the system that actually stores data. For example, flat file
* DBMS is the system that provides query, data integraty, concurrency control and durability.
* mongo, redshift, sqllite are DBMS

### 1.2 physical layer and logical layer
* physical means hardware, disk etc.
* logical means query, insert, delete etc.
* Database emphysizes the separation of physical layer and logical layer

### 1.3 data models: we will learn relational data model
* relational data model: redshift, sqllite
* document data model: mongoDB
* key/value: redis
* graph: neo4j
* column-family: HBase
* Array/matrix: machine learning

### 1.4 relational data model concepts
#### 1.4.1 primary key and foreign key
#### 1.4.2 relational algebra vs relational calculas
* relational algebra is procedural meaning it defines the procedure of how the data is retrieved/updated.
* relational calculas is non-procedural meaning it does **NOT** define the procedure of how the data is retrieved/updated.
* sql is non-procedural

#### 1.4.3 relational algebra vs relational calculas
* select (predicate) -> -> where clause
* projection -> select clause
* union -> union
* intersect -> intersect
* difference -> except
* product -> from/from cross_join
* join -> natural_join/from where

## 2. disk/storage manager
### 2.1 what it is? 
* it is the system that manages how the data is fetched from disk to memory.

### 2.2 mmap
* mmap maps the content of a file into a process's address space. OS is responsible for moving the data in and out of physical memory.
* This works for read-only access. It is complicated when there are multiple writers.
  * flush dirty pages to disk in the correct order??
  * specialized prefetching
  * buffer replacement policy -> LRU??
  * thread/process scheduling
* Some solutions: 
  * madvise: tell OS how you expect to read certain pages.
  * mlock: tell OS that memory ranges can not be paged out.
  * msync: tell OS to flush memory ranges out to disk.

### 2.3 file
* File is just OS file

* There are different ways that DBMS manages pages in the files on disk
 * heap file organization
 * sequential / sorted file organization
 * hashing file organization
* Heap file organization: two ways
 * linked list: free page list and data page list. Looks like malloc and free mechanism learned from OS
 * page directory: One special page which contains pointer to differetn data page. This page tracks the location of data pages. This page is called "directory". The directory also records the number of free slots per page.

### 2.4 page
* A page is fixed-size block of data.
 * it can contain tuples, meta-data, indexes, log records.
 * each page is given a unique identifier, i.e pageId.
 * DBMS keeps indirection layer to map page ids to physical locations.

* The begin part of page is Header
 * page size, checksum, DBMS version, transaction visibility etc

* There are approaches for the storage architecture:
 * tuple-oriented
 * log-strctured

#### 2.4.1 page's tuple storage approach
* A naive solution is to append new tuple to the end of page. But it is hard to delete tuple and maintain it.
* The most common layout scheme is "slotted pages".
* After header, it is a section called "slot array". It maps "slot" to the tuple's start position offset
* The "slot array" grows from start of page to end, while tuples grows from end of page to start.

#### 2.4.2 page's log-structured organization
* Instead of storing data. We store the logs, e.g, insert, update, delelte logs
* To read a record, the DBMS scans the log backwards to "re-creates" the tuple to find what it needs.
* build index to allow it to jump to locations in the log.
* There is periodically compaction running.