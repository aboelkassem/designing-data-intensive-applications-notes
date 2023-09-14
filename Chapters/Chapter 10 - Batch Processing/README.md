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
