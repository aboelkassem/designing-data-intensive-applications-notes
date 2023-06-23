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

# Acknowledgements
I would like to express my appreciation to Martin Kleppmann for authoring "Designing Data-Intensive Applications" and sharing a wealth of knowledge with the community. My notes are derived from the concepts and ideas presented in his book.

# License
The content in this repository is provided under the [MIT License](https://opensource.org/license/mit). You are free to use, modify, and distribute the notes for personal educational purposes.

# Feedback and Support
If you have any feedback, suggestions, or questions about my personal notes, please feel free to reach out to me. Your support and encouragement are greatly appreciated.
