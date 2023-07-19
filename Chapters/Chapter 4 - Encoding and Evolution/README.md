## Chapter 4: Encoding and Evolution
## Table of content


Changes to an application’s features also requires a change to data that it stores: perhaps a new field or record type needs to be captured (also, changes on Database schema).

In a large application, and this application is deployed in multiple servers, the code changes often cannot happen instantaneously (at no time):

- With ***server-side*** applications you may want to perform **a rolling upgrade** which new version to a few nodes/services at a time.
- With client-side applications you’re at the mercy of the user, who may not install
the update for some time.

So, this will cause a compatibility issue whether between client/server or between the services (in the world of microservices/SOA architecture) or even producers/consumers different version.

To solve this issue, your system must support the two types of data compatibility:

- **Backward compatibility**
    - Newer code can read data that was written by older code (for example, you can open docx files written by Office 2010 with Office 2019).
    - This is not a problem, because New code already knows the format old data, so it can handle it.
- **Forward compatibility**
    - Older code can read data that was written by newer code (for example, your mobile app is no updated with the new version of API).
    - Can be tricky in real world, because it requires older code to ignore additions made by a
    newer version of the code. Take only needed/matters.

## Data Representation

Programs usually work with data in (at least) two different representations:

- **In Memory**: objects, structs, lists, arrays, hash tables, trees, an so on. These data structures are optimized for efficient access and manipulation by the CPU (typically using pointers).
- **Persisted**: In file or network or IPC. you have to **encode** it as some kind of self-contained **sequence of bytes** which we call **data format**. Since a pointer wouldn’t make sense to any other process.

Thus, we need some kind of translation between the two representations. The translation from the in-memory representation to a byte sequence is called **encoding or serialization,** and the reverse operation called decoding or deserialization.

## Schema

Which is a collection of fields, each one have Name and datatype.

- **Schema on Read**: The code infers the schema while reading.
    - Example: Document Databases like MongoDB
- **Schema on Write**: the code enforce the schema while writing.
    - Example: Relational Databases like MySQL, Postgres
