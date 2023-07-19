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
## Formats for Encoding Data

Like JSON, XML, and Binary Variants. This formats is standard encoding between any programming language.

### JSON

JSON’s popularity is mainly due to its built-in support in web browsers and human-readable.

Support for Unicode character strings (human readable text), but they don’t support binary strings (sequences of bytes without a character encoding)

Support backward and forward compatibility

the following format we will use it in other different formats

```jsx
{
	"userName": "Martin",
	"favoriteNumber": 1337,
	"interests": ["daydreaming", "hacking"]
}
```

### XML

XML is often criticized for being too verbose and unnecessarily complicated

Support for Unicode character strings (human readable text), but they don’t support binary strings (sequences of bytes without a character encoding)

Support backward and forward compatibility:

### BSON

Binary representation of JSON-like documents.

Designed for efficient storage and retrieval of documents in MongoDB (a NoSQL database).

### MessagePack

Doesn’t support backward and forward compatibility.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%204%20-%20Encoding%20and%20Evolution/images/message-pack.png" width="700" hight="500"/>
</p>

### Thrift and Protocol Buffer

Protocol Buffer Developed at Google while Thrift Developed at Facebook

Both Support backward and forward compatibility

Both Require a schema for any data that is encoded

```protobuf
message Person {
	required string user_name = 1;
	optional int64 favorite_number = 2;
	repeated string interests = 3;
}
```

Thrift has two different encoding format **BinaryProtocol** and **CompactProtocol**

**BinaryProtocol** 

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%204%20-%20Encoding%20and%20Evolution/images/thrift-protocol.png" width="700" hight="500"/>
</p>

The difference between this format and above format (Message Pack) is that there are no field names (userName, favoriteNumber, interests). Instead, the encoded data contains **field tags**, which are numbers (1, 2, and 3). Those are the numbers that appear in the schema definition. Field tags are like **aliases** for fields.

**CompactProtocol**

Like BinaryProtocol but it just packs the same information into only 34 bytes by **merging** the field type and tag number into a single byte. Rather than using a full eight bytes for the number 1337.

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%204%20-%20Encoding%20and%20Evolution/images/thrift-compact-protocol.png" width="700" hight="500"/>
</p>

**ProtocolBuffers**

<p align="center" width="100%">
  <img src="https://github.com/aboelkassem/designing-data-intensive-applications-notes/blob/main/Chapters/Chapter%204%20-%20Encoding%20and%20Evolution/images/protocol-buffers.png" width="700" hight="500"/>
</p>

**How Thrift and Protocol Buffers handle schema changes while keeping backward and forward compatibility ?**

It depends on the tag field, If a field value is not set, it is simply deleted from the encoded record.

In Forward compatibility, If old code (which doesn’t know about the new tag numbers you added) tries to read data written by new code, including a new field with a tag number it doesn’t recognize and ignore that field. **just read what needed data.**

In backward compatibility, As long as each field has a **unique tag number**, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it **required.** So every field you add after the initial deployment of the schema must be **optional or have a default value**
