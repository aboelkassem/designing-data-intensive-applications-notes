# Chapter 10: Batch Processing
All systems can fit into three main categories:

- **Services (online)**, where it waits for requests from a client to arrive, tries to handle them as quickly as possible, and sends back a response.
- **Batch Processing Systems (offline)**, takes a large amount of input data where it runs a job to process a large amount of *bounded* input, and produce some output data. It is often scheduled periodically, and can take up to several days as no user is typically waiting for it to finish.
- **Stream Processing Systems (near-real-time)**, Stream processing is somewhere between online and offline/batch processing, where is consumes *unbounded* input/events shortly after its available, processes it, and produces output.  It build upon batch processing and have lower latency.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%2010%20-%20Batch%20Processing/images/types-of-systems.png" width="700" hight="500"/>
</p>

In this chapter, we will look at MapReduce and several other batch processing algorithms and frameworks.
