- [1. ExtendibleHashMap](#1-extendiblehashmap)
  * [1.1 Find](#11-find)
  * [1.2 insert](#12-insert)
    + [1.2.1 peroform find](#121-peroform-find)
    + [1.2.2 insert](#122-insert)
    + [1.2.3 split the Bucket](#123-split-the-bucket)
      - [1.2.3.1 double global slot](#1231-double-global-slot)
      - [1.2.3.1 update global slot to point to new Bucket](#1231-update-global-slot-to-point-to-new-bucket)
- [2. LRU replacer](#2-lru-replacer)
- [3. Buffer pool manager](#3-buffer-pool-manager)
  * [3.1 components](#31-components)
  * [3.2 DiskManager](#32-diskmanager)
    + [3.2.1 write page](#321-write-page)
    + [3.2.2 read page](#322-read-page)
  * [3.3 NewPage](#33-newpage)
    + [3.3.1 LRU Replacer](#331-lru-replacer)
  * [3.4 FetchPage](#34-fetchpage)
    + [3.4.1 If the page is aleady in the buffer pool.](#341-if-the-page-is-aleady-in-the-buffer-pool)
    + [3.4.2 If the page is not in the buffer pool](#342-if-the-page-is-not-in-the-buffer-pool)
  * [3.5 UnpinPage](#35-unpinpage)
  * [3.6 DeletePage](#36-deletepage)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. ExtendibleHashMap
## 1.1 Find
* hash the key
* use global slot as mask to find offset of global slot
* global slot will point to a ```Bucket``` with a list of ```Elements``` that stored in the bucket
* iterate the element to find if the given key is stored in the Bucket

## 1.2 insert
### 1.2.1 peroform find
* try find the given key to see if it is already inside the hashtable.
* If exist overwrite the value and return

### 1.2.2 insert
* If the key does not exist before, create a new Element and push it into the correspond Bucket based on hashkey and mask

### 1.2.3 split the Bucket
* If the element in the Bucket exceed the given max capacity, we need to split the Bucket.
* We first create a new Bucket. Then we redistribute the Elements between old Bucket and new Bucket based on their hashKey and mask result.
* Increase the localDepth. If localDepth > globalDepth, we need to increase the globalDepth which means we need to double the global slot.
* Update the global slot to point to new Bucket.
* It is possible that by increasing localDepth by 1, all elements still are hashed to the same Bucket. Thus, we need to do this recursively by continuously increasing the localDepth, untill at least one Element is hashed to new Bucket.

#### 1.2.3.1 double global slot
* The way to double the global slot is by duplicating existing Bucket in order.
* The reason we can do this is because they symetric property of Bucket location. With globalDepth increased by 1, it is possible the hashKey be masked to the second half of the new global slot. But since it is symetric location, it still points to the original Bucket.

```
if (bucket->localDepth > globalDepth) {
	LOG_DEBUG("we need to increase globalDepth");
	int size = buckets.size();
        
	for (int i = 0; i < size; i++) {
		buckets.push_back(buckets[i]);
    }

    globalDepth = bucket->localDepth;
}
```

#### 1.2.3.1 update global slot to point to new Bucket

* Note since we may need to increase the localDepth by multiple times which means we can duplicate the Buckets for multiple times, we need to scan through all global slots to update the pointer of new Bucket.

```
		for (size_t i = 0; i < buckets.size(); i++) {
            if (buckets[i] == bucket) {
                if ((i & mask) == oldOffSet) {
                    buckets[i] = bucket;
                } else {
                    buckets[i] = newBucket;
                }
            }
        }
```

# 2. LRU replacer

* LRU replacer is pretty simple, as lintcode problem
* Just use ```unordered_map``` and ```linkedList``` as implementation.
* In c++ ```LinkedList``` is ```std::list```. Some of member functions are useful
 * ```pop_back()```
 * ```push_front()```
 * ```erase()```
 * etc

# 3. Buffer pool manager
* The buffer pool manager manages a list of Pages in memory which can be read and write by different threads.
* It also talks to DiskManger to write the Page data back into disk.

## 3.1 components
* DiskManager
* PageTable
* FreeList
* LRUReplacer
* Pages

## 3.2 DiskManager
* This implementation does not use ```Page directory```. There is no such "Page" that stores information about the mapping from a ```pageId``` to a ```disk offset```.
* Rather, it holds a const ```PAGE_SIZE``` and ```page_id``` starts from 0 and increament accordingly.
* Since Pages are appending directly to the file, it can use ```page_id * PAGE_SIZE``` to figure out the offset.
* it uses ```std::fstream``` to open/write the file on disk

### 3.2.1 write page
* ```void DiskManager::WritePage(page_id_t page_id, const char *page_data)```
* use ```page_id * PAGE_SIZE``` to figure out the offset
* seek to the offset ```db_io_.seekp(offset);```
* write the whole ```PAGE``` by ```db_io_.write(page_data, PAGE_SIZE);```
* ```db_io_.flush();```

### 3.2.2 read page
* seek the offset
* read the page by ```db_io_.read(page_data, PAGE_SIZE);```

## 3.3 NewPage
* Try to get a free Page from ```free_list```. If ```free_list``` is not empty, we need to get it from LRUReplacer.
* If neither way get the free page, ```newPage``` fails.
* If we are retriving free page from LRU replacer, it means we need to evict a page out of buffer pool. This requires us to remove the evicted page's ```pageId``` out of ```pageTable```.
* We fetch a ```pageId``` from ```DiskManager``` by calling ```AllocatePage()```.
* Associate the ```pageId``` with the new page with pageTable.
* pin the page.

### 3.3.1 LRU Replacer
* The page can be pined meaning it is currently used by other thread. This means we are not able to evict it out of buffer pool manager.
* Thus only the page that are not pined can be added into LRUReplacer.
* If the page to be evicted is dirty, we need to write it back to disk using ```DiskManager``` before evicting. And clean the dirty flag.
* Note there is no reordering of pages happening inside ```LRU```, as the only operations that supported are ```erase```, ```insert``` and ```victim```. There is nothing like ```touch``` in ```LRU```. I think this also has something to do with how ```FetchPage``` is implemented. ```FetchPage``` means we can either read or write to the page. There is no "read-only" operation supported inside buffer pool manager. This is also the reason that "read/write" lock is not needed for buffer pool manager.

## 3.4 FetchPage
* ```FetchPage(page_id_t page_id) ```
* The page to be fetched can be either in Buffer Pool or on disk. 
* If the page is in the buffer pool, the ```pageId``` is in the page table. 
* From this we know, if a page is evicted out of buffer pool, we need to remove it from page table to maintain the **invariance**.

### 3.4.1 If the page is aleady in the buffer pool.
* We can get the page from page Table.
* If the page is in ```LRU replacer```, we need to remove it from ```LRU Replacer```, since the fetched page will be put into ```pined``` state. 
* This can prevent us from evicting it out of Buffer pool later.

### 3.4.2 If the page is not in the buffer pool
* If the page is on disk, we need to find a new FreePage to hold it. This is the same process as calling ```NewPage```. 
* Read the data from disk by passing given ```pageId``` and free Page to ```DiskManager```.
* We need to put ```<pageId, Page>``` into pageTable and ```pin``` the page.

## 3.5 UnpinPage
* decrement pin count
* if ```pin count == 0```, move the page to ```LRU replacer``` for eviction.

## 3.6 DeletePage
* only perform when no pin count
* clean page by clearing dirty flag, reset memory, unset pageId
* remove pageTable
* remove from LRU
* add page back to free list.