# Designing Data Intensive Applications Notes
This repository serves as a collection of my own notes and summaries for studying and understanding the concepts covered in the book.

## About the Book
"Designing Data-Intensive Applications" by Martin Kleppmann is a highly regarded resource for developers, engineers, and architects working with data-intensive systems. The book explores the principles, patterns, and best practices for designing and building robust, scalable, and reliable applications that process and store large volumes of data.

The book is available [here](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063).


# Table Of Content

# Part I: Foundation of Data Systems
## Chapter 3: Storage and Retrieval
## Data Structures That Power Your Database

Imagine having the world's simplest database at your disposal, implemented using just two Bash functions. This minimalist yet functional database allows you to store and retrieve key-value pairs effortlessly. Here's the code snippet that brings this database to life:

```bash
#!/bin/bash
db_set () {
    echo "$1,$2" >> database
}
db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

This two functions: Append only writes and For lookups read O(n)

**An index** is an additional structure that is derived from the primary data (metadata)
## Log Structured Storage (Hash Index)

Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote

![](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/log-structured-storage.png?raw=true)

Append only Data structure. The log file is sequentially written, which allows for fast write operations since it avoids random disk seeks.

What if the disk is full or out of space? we will group the logs into segments with size

![](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/hash-index-compaction.png?raw=true)

New records are appended to a *segment* of certain size which is being merged and compacted by a background thread. 

While it is going on, we continue read and write using the old segment files until the merging process is complete, we switch read requests to using the new merged segment instead of the old segments and finally the old segments are deleted.

### The issues of this index and how to solve them
- **File format**: CSV is not the best format for a log, It’s faster and simpler to use a binary format
- **Deleting records**: Use special deletion record called tombstone, just mark this data with deleted flag and the merging process to discard it.
- **Crash recovery:** If the database is restarted, the in-memory hash maps are lost, so we save a snapshot of each segment’s hash map on disk.
- **Concurrency control:** Use one thread for writing and multiple threads for reading
- **Partially written records**: to prevent corrupted data, we add checksum

### The Props of Log structured hash index
- Sequential write operations is faster than than random writes
- Concurrency and crash recovery are much simpler, the system can replay the log file to restore the database to a consistent state.
- No data fragmentation (meaning the data is stored distributed on the hard disk)

### The limitations of Log structured hash index
- The hash table must fit in memory (for fast read/writes), so you have a space issue
- Range queries are not efficient

## SSTable (Stored String Tables)
To Fix the memory space issue, we will use this SSTable

This work by sorting the keys, which will make the keys unique after merge process
- Each key will appear once in the segment
- Each segment will be sorted by key
- Keep separate hashtable for each segment
- Merge each segments into one segment like the following
    - you start reading the input files side by side, look at the first key in each file, copy the lowest key (sorted) to new merged segment and repeat.

![](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/sstable-compaction.png)

Then, The hash map index will contains the first key of each segment instead of all keys
![](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/sstable-compression.png)

- Each segment will be sorted by key
- Keep separate hashtable for each segment

How to save the segments sorted on memory, Our incoming writes can occur in any order? We will use **AVL Tree.**

The flow to save data

- When a write comes in, add it to an in-memory balanced tree data structure called **memtable**
- When the memtable gets bigger than some threshold (for example > 100 MB), write it out to disk as an SSTable file (Segment) which is the most recent segment
- In order to serve a read request, **first try to find the key in the memtable**, then in the most recent on-disk segment, then in the next-older segment, etc
- From time to time, **run a merging and compaction process** in the background to
combine segment files and to discard overwritten or deleted values

![](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/sstable-avl-tree.png)

This algorithm (named also LSM Tree) used in LevelDB, RockDB, Cassandra, HBase and Lucene engine (used in Elastic Search and Solr) 
LSM-Trees offer an efficient segment merging mechanism, eliminate the need for an in-memory index for all keys, and support grouping and compressing records before writing to disk, but they can be slow for non-existent key lookups (which can be improved with a bloom filter).

## B-Tree
This is the most widely used indexing in Relational Databases like Postgres. Like SSTables, B-trees keep key-value pairs sorted by key which allows efficient keyvalue lookups and range queries.

Unlike SSTable (which break the database into segments), B-Tree break the database into fixed Size blocks (pages). Each page is have size (traditionally 4 KB in size)

Unlike SSTable (which is stored on memory), B-Tree is in the disk

B-Tree is balanced, so `n` keys always have depth of `O(log n)`. It also have a branching factor of several hundreds typically.

![Untitled](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/B-Tree.png)

If you want to update the value for an existing key in a B-tree, you search for the leaf page containing that key, change the value in that page, and write the page back to disk.

If you want to add a new key, you need to find the page whose range encompasses the new key and add it to that page.

- If there isn’t enough free space in the page to accommodate the new key, it is **split** into two half-full pages, and the parent page is updated to account for the new subdivision of key ranges

![Untitled](https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/images/B-Tree%20Insertion.png)

> Most databases can fit into a B-tree that is **three or four levels** deep, so you don’t need to follow many page references to find the page you are looking for. (A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.)

### B-Tree Reliability
In order to make the database resilient to crashes

- B Tree uses **Write AHead Log (WAL)** ⇒ This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself.

To Improve the previous way for performance, 
- They use new way called **Copy On Write** scheme, Any new data written to a different location, and a new version of the parent pages in the tree is created, pointing at the new location.
- Another optimization way is to use short key names (B-Tree+)
- Another optimization way is to use Extra pointers for each node to point previous and next nodes

# Acknowledgements
I would like to express my appreciation to Martin Kleppmann for authoring "Designing Data-Intensive Applications" and sharing a wealth of knowledge with the community. My notes are derived from the concepts and ideas presented in his book.

# License
The content in this repository is provided under the [MIT License](https://opensource.org/license/mit). You are free to use, modify, and distribute the notes for personal educational purposes.

# Feedback and Support
If you have any feedback, suggestions, or questions about my personal notes, please feel free to reach out to me. Your support and encouragement are greatly appreciated.
