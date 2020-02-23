# Buffer pool manager

- [Buffer pool manager](#buffer-pool-manager)
  * [1. General database concepts](#1-general-database-concepts)
    + [1.1 database vs database management system](#11-database-vs-database-management-system)
    + [1.2 physical layer and logical layer](#12-physical-layer-and-logical-layer)
    + [1.3 data models: we will learn relational data model](#13-data-models--we-will-learn-relational-data-model)
    + [1.4 relational data model concepts](#14-relational-data-model-concepts)
      - [1.4.1 primary key and foreign key](#141-primary-key-and-foreign-key)
      - [1.4.2 relational algebra vs relational calculas](#142-relational-algebra-vs-relational-calculas)
      - [1.4.3 relational algebra vs relational calculas](#143-relational-algebra-vs-relational-calculas)
    + [1.5 OLTP vs OLAP](#15-oltp-vs-olap)
    + [1.5.1 N-ary storage model (NSM)](#151-n-ary-storage-model--nsm-)
    + [1.5.2 Decomposition Storage model (dsm)](#152-decomposition-storage-model--dsm-)
  * [2. disk/storage manager](#2-disk-storage-manager)
    + [2.1 what it is?](#21-what-it-is-)
    + [2.2 mmap](#22-mmap)
    + [2.3 file](#23-file)
    + [2.4 page](#24-page)
      - [2.4.1 page's tuple storage approach](#241-page-s-tuple-storage-approach)
      - [2.4.2 page's log-structured organization](#242-page-s-log-structured-organization)
  * [3. buffer pool](#3-buffer-pool)
    + [3.1 page table and frame](#31-page-table-and-frame)
      - [3.1.1 page table vs page directory](#311-page-table-vs-page-directory)
      - [3.1.2 page table and meta-data](#312-page-table-and-meta-data)
      - [3.1.3 page table and hash](#313-page-table-and-hash)
    + [3.2 replacement policy](#32-replacement-policy)
    + [3.3 multiple buffer pools](#33-multiple-buffer-pools)
  * [4. Extendible hashing](#4-extendible-hashing)
    + [4.1 extendible vs chained hashing](#41-extendible-vs-chained-hashing)
      - [4.2 internal data structure](#42-internal-data-structure)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

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

### 1.5 OLTP vs OLAP
* OLTP: Simple queries that read/update a small amount of data that is related to a single entity in the database.
* OLAP: Complex queries that read large portions of the database spanning multiple entities.

### 1.5.1 N-ary storage model (NSM)
* stores all attributes for a single tuple continguously in a page
* ideal for OLTP workloads where queries tend to operate only on an individual enitty and insert heavy workloads.

### 1.5.2 Decomposition Storage model (dsm)
* This is known as a "column store"
* This model is ideal for OLAP workloads where read-only queries perform large scan over a subset of the table's attributes.
* There is meta-data stored to help restoring the whole row.


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

## 3. buffer pool
### 3.1 page table and frame
* The buffer pools consists of two parts: frame and page table. Page table maintains the pageId to frame. Frame contains the actual page data.
* frame: Inside the buffer pool, the place where we store the page data is called frame.
* page table: page table is the table map from pageId to frame which will be used by execution engine to query the page. If the page table says it is not in frame, we will need to bring to it from disk.

#### 3.1.1 page table vs page directory
* page directory is map from page id to page location on disk, i.e, which file and what offset.
* page table is map from page id to frame.

#### 3.1.2 page table and meta-data
* The page table is considered as meta-data of buffer pool. Inside the page table, there are two meta-data that needs to be kept track of, i.e the dirty flag and pin.
* use pin to mark the certain page table entry as is being read by some threads. This will prevent other threads to evict the page entry.
* dirty flag: mark the page in buffer pool has been modified and we need to make sure it is writen back to disk.

#### 3.1.3 page table and hash
* static hashing: means that we know all the keys we want to hash before hand. We do not need to delete and update the key/value. Example: linear probe hashing, Robin hood hashing, cuckoo hashing
* dynamic hashing: dynamic hash tables are able to grow and shrink on demand. Example, Extendible hashing and Chained hashing


### 3.2 replacement policy
* LRU

### 3.3 multiple buffer pools
* buffer pool per database instance
* buffer pool per page type: index page, tuple pages
* Thus, we do not need thread-safe guarentee.

## 4. Extendible hashing
### 4.1 extendible vs chained hashing
* Chained hashing is how JDK hashmap is implemented
* extension of chained hashing, the difference is that the chained hash will increment the linked list of buckets to solve collision. But the extendible hashing tries to keep the bucket size the same so that we do not have to do linear scan, when collision happens.
* The basic idea is to create new bucket, when the old one is full.

#### 4.2 internal data structure
* There are two data structures the extendible hash table maintain: **global slots** and **local buckets**.
* The globle slot is a pointer which points to a local bucket.
* There are two depths the extendible hashing maintain: **global depth** and **local depth**. 
 * The hash table first hash the key and then use global depth to fetch the bits from the hashed key to find the cooresponding global slot. 
 * Thus, we can see the global depth bit controls which local bucket to go to search.
* the local depth bit represents what global slots are referening to this local bucket. Since local depth can be shorter than global depth, it means there are more than two global slots that are referencing this local buckets
* Note global depth equals the **largest** number of local depth.
