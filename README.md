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

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6d18ac3-bd51-485b-a98c-eec966603ceb/Untitled.png)

Append only Data structure. The log file is sequentially written, which allows for fast write operations since it avoids random disk seeks.

What if the disk is full or out of space? we will group the logs into segments with size

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/364d41b5-0d52-4963-8f70-8c111c7adb23/Untitled.png)

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

# Acknowledgements
I would like to express my appreciation to Martin Kleppmann for authoring "Designing Data-Intensive Applications" and sharing a wealth of knowledge with the community. My notes are derived from the concepts and ideas presented in his book.

# License
The content in this repository is provided under the [MIT License](https://opensource.org/license/mit). You are free to use, modify, and distribute the notes for personal educational purposes.

# Feedback and Support
If you have any feedback, suggestions, or questions about my personal notes, please feel free to reach out to me. Your support and encouragement are greatly appreciated.
