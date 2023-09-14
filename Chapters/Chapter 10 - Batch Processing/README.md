# Chapter 10: Batch Processing
All systems can fit into three main categories:

- **Services (online)**, where it waits for requests from a client to arrive, tries to handle them as quickly as possible, and sends back a response.
- **Batch Processing Systems (offline)**, takes a large amount of input data where it runs a job to process a large amount of *bounded* input, and produce some output data. It is often scheduled periodically, and can take up to several days as no user is typically waiting for it to finish.
- **Stream Processing Systems (near-real-time)**, Stream processing is somewhere between online and offline/batch processing, where is consumes *unbounded* input/events shortly after its available, processes it, and produces output.  It build upon batch processing and have lower latency.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2010%20-%20Batch%20Processing/images/types-of-systems.png" width="700" hight="500"/>
</p>

In this chapter, we will look at MapReduce and several other batch processing algorithms and frameworks.

## Batch Processing with Unix Tools

Say you have a web server that appends a line to a log file every time it serves a request.

```
216.58.210.78 - - [27/Feb/2015:17:55:11 +0000] "GET /css/typography.css HTTP/1.1" 200 3377 "http://martin.kleppmann.com/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.115 Safari/537.36"
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"
```

Many data analysis can be done in few minutes using some combination of Unix commands `awk`, `sed`, `grep`, `sort`, `uniq`, and `xargs`.

for example the following command, can find the five most popular pages on your website

```
cat /var/log/nginx/access.log |
awk '{print $7}' |
sort |
uniq -c |
sort -r -n |
head -n 5
```

- Line 1: read the log file
- Line 2: Split each line into fields by whitespace, take the seventh field ⇒ like string.split(” ”)[6] in c# …. awk is scripting language
- Line 3: Sort the list alphabetically.
- Line 4: Filter out repeated lines, -c print the counter for every distinct URL.
- Line 5: Sort by the number (-n which is repeated times) in reverse order (-r)
- Line 6: take the first five lines

The output of command will be like

```
4189 /favicon.ico
3631 /2013/05/24/improving-security-of-ssh-private-keys.html
2124 /2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html
1369 /
915 /css/typography.css
```

**A uniform interface**

A compatible interface. If you want to be able to connect any program’s output to any program’s input, that means that all programs must use the same input/output interface.

In Unix, that interface is a file (or, more precisely, a file descriptor). A file is just an ordered sequence of bytes. Because that is such a simple interface, can be an actual file on the filesystem, a communication channel to another process (Unix socket, stdin, stdout), a device driver (say /dev/audio or /dev/lp0), a socket representing a TCP connection, and so on.

By convention, many Unix programs treat this sequence of bytes as ASCII text. The above example treat their input file as a list of records separated by the \n (newline, ASCII 0x0A).

The good thing is that interface give the a shell user to wire up the input and output in whatever way they want; you can even write your own program/tool (just need follow the interface by reading input from **stdin** and writing the output to **stdout**)

Reasons that Unix tools are so successful includes that the input files are treated as immutable, we can end the pipeline are any point, and we can write the output of intermediate stages to files for fault-tolerance.

The biggest limitation of Unix tools is that it can only run on a single machine, that's where tools like Hadoop come in.

## MapReduce and Distributed Filesystems

MapReduce is a bit like Unix tools, but distributed across potentially thousands of machines. While Unix tools use stdin and stdout as input and output, MapReduce jobs read and write files on a distributed filesystem. In Hadoop’s implementation called HDFS (Hadoop Distributed File System).

Various other distributed **filesystems** besides HDFS exist, such as GlusterFS and the Quantcast File System (QFS). **Object storage** services such as Amazon S3, Azure Blob Storage, and OpenStack Swift are similar in many ways.

There is two approaches for storing files

- **Network Attached Storage (NAS) and Area Network (SAN) architectures**: implemented by a centralized storage appliance (hardware or Fibre channel).
- **Shared-Nothing principle (horizontal scaling)**: requires no special hardware, only computers connected by a conventional datacenter network.

HDFS is based on the shared-nothing principle. Which consists of a daemon process running on each machine exposing a network service that allows other nodes to access files stored on that machine. A central server called the **NameNode** keeps track of which file blocks are stored on which machine. In order to tolerate machine and disk failures, file blocks are replicated on multiple machines.

### MapReduce Job Execution

MapReduce is a programming framework with which you can write code to process large datasets in a distributed filesystem like HDFS.  The previous example of Unix log analysis, this how data processing in MapReduce.

1. Read a set of input files, and break it up into records. In the web server log example, each record is one line in the log (that is, \n is the record separator).
2. Call the **mapper function** to extract a key and value from each input record. In the preceding example, the mapper function is awk '{print $7}': it extracts the URL ($7) as the key, and leaves the value empty.
3. Sort all of the key-value pairs by key. In the log example, this is done by the first sort command.
4. Call the **reducer function** to iterate over the sorted key-value pairs. If there are multiple occurrences of the same key, the sorting has made them adjacent in the list, so it is easy to combine those values without having to keep a lot of state in memory. In the preceding example, the reducer is implemented by the command uniq -c, which counts the number of adjacent records with the same key

Those four steps can be performed by one MapReduce job. Steps 2 (map) and 4 (reduce) are where you write your custom data processing code. Step 1 (breaking files into records) is handled by the input format parser. Step 3, the sort step, is implicit in MapReduce—you don’t have to write it, because the output from the mapper is always sorted before it is given to the reducer.

To create a MapReduce job, you need to implement two callback functions, the mapper and reducer, which behave as follows:

- Mapper: called once for every input record, its job is to extract the key and value from the input record.
- Reducer: takes the key-value pairs produced by the mappers, collects all the values belonging to the same key, and use them to produce a number of output records.

The role of the mapper is to prepare the data by putting it into a form that is suitable for sorting, and the role of the reducer is to process the data that has been sorted

MapReduce can parallelize a computation across many machines. The mapper and reducer only operate on one record at a time; they don’t need to know where their input is coming from or their output is going to.

The following diagram shows the dataflow in Hadoop MapReduce job. The input is a directory in HDFS and each file is separate **partition** that can be processed by a separate map task (m1, m2, m3)

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2010%20-%20Batch%20Processing/images/map-reduce.png" width="700" hight="500"/>
</p>

Application code (e.g., JAR files if java application) is run in the map task and the MapReduce framework first copy this code to the appropriate machines, then starts the map task and begins reading the input file, passing one record at a time to the mapper callback. The output of the mapper consists of key-value pairs. 

The key-value pairs must be sorted, but the dataset is likely too large, Instead, the sorting is performed in stages. First, each map task **partitions** its output by reducer, based on the hash of the key. Each of these partitions is written to a sorted file on the mapper’s local disk. The reducers connect to each of the mappers and download the files of sorted key-value pairs for their partition.

The reduce task takes the files from the mappers and merges them together, preserving the sort order. Thus, if different mappers produced records with the same key, they will be adjacent in the merged reducer input

MapReduce jobs cannot have any randomness, and the range of problems we can solve with single MapReduce job is limited, so commonly jobs are chained together to form a *workflow* (output of one job becomes the input to the next job), Hadoop MapReduce framework does not have any particular support for workflows, so it done implicitly using directory names.
