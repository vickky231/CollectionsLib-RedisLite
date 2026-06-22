# DataForge & Redis Lite – Design Proposal

## Project Overview

DataForge is a custom C++ collections library consisting of three core data structures: **DynamicArray**, **LinkedList**, and **HashMap**.
These structures are used to build **Redis Lite**, a simplified in-memory key-value database that supports efficient storage, retrieval, and deletion of data.

---

## Public API

### DynamicArray

```cpp

Function	Complexity
pushBack	O(1) amortized
popBack	  O(1)
get	      O(1)
set	      O(1)
resize	  O(n)                    

```
purpose

HashMap requires a bucket table whose size can grow dynamically during execution.
Data Members
T* data;
int size;
int capacity;
•	data stores the address of heap memory. 
•	size stores the number of elements. 
•	capacity stores allocated memory slots. 
Resizing Strategy
When:
size == capacity
capacity is doubled:
4 → 8 → 16 → 32

### LinkedList

```cpp

Function	Complexity
insert	    O(1)
search	    O(n)
remove	    O(n)
update	    O(n)

```
Purpose

Stores key-value pairs and is used for collision handling within HashMap buckets..

Node Structure

struct Node
{
    string key;
    string value;
    Node* next;
};
Memory Layout
     head
      |
      v
[key1:value1]
      |
      v
[key2:value2]
      |
      v
     NULL


### HashMap

```cpp

                                                                                              functions         	Complexity
put	                O(1) average	
get	                O(1) average
remove	            O(1) average
clear	              O(n)
rehash	            O(n)

"O(1) average" means that on average (most of the time) the operation takes a constant amount of time, regardless of how many elements are stored.


```

Purpose

Provides efficient implementation of:
SET
GET
DEL
EXISTS
Components
•	Dynamic Array 
•	Linked List 
•	Hash Function 
•	Rehashing 

Memory Layout
Bucket[1]

[name:Vivek]
      |
      v
[city:Karnal]

Provides fast key-based access and serves as the primary storage structure for Redis Lite.

---

## Redis Lite

Redis Lite is a command-line key-value store built using the DataForge library.

### Supported Commands

```text
SET key value
GET key
DEL key
EXISTS key
SIZE
CLEAR
EXIT
```

### System Architecture

```text
                                    +------------------+
                                    |    RedisLite     |
                                    +------------------+
                                              |
                                              v
                                  +------------------------+
                                  | SuperCollectionLibrary |
                                  +------------------------+
                                     /          |         \ 
                                    /           |          \
                                  v             v            v

                        +--------------+ +--------------+ +--------------+
                          | DynamicArray | | LinkedList   | |  HashMap  |
                        +--------------+ +--------------+ +--------------+
                                                                   |
                                                                   v
                                                              +------------+
                                                              |  HashNode  |
                                                              +------------+
Components
RedisLite
•	Main application that processes commands like SET, GET, and DELETE. 
SuperCollectionLibrary
•	A collection of reusable data structures used by RedisLite. 
DynamicArray
•	Stores the HashMap buckets. 
•	Provides fast access using indexes. 
LinkedList
•	Handles collisions when multiple keys map to the same bucket. 
•	Connects nodes together. 
HashMap
•	Main data structure for storing key-value pairs. 
•	Uses hashing for fast insertion, search, and deletion. 
HashNode
•	Stores a single key-value pair. 
•	Contains a key, value, and pointer to the next node.

```

The HashMap stores buckets in a DynamicArray and resolves collisions using LinkedLists.

---

## Internal Representation and Rule of Three

All data structures manage dynamic memory and therefore implement:

* Destructor
* Copy Constructor
* Copy Assignment Operator

### DynamicArray

* Uses a dynamically allocated array.
* Destructor releases memory using `delete[]`.
* Copy operations perform deep copies.

### LinkedList

* Stores nodes containing key-value pairs.
* Destructor deletes all nodes.
* Copy operations create new nodes.

### HashMap

* Stores buckets and collision chains.
* Destructor frees all buckets and linked lists.
* Copy operations create independent copies.

Deep copying prevents shared ownership, dangling pointers, and double-free errors.

---

## Complexity Analysis

| Structure    | Operation | Average        | Worst |
| ------------ | --------- | -------------- | ----- |
| DynamicArray | get()     | O(1)           | O(1)  |
| DynamicArray | append()  | O(1) amortized | O(n)  |
| DynamicArray | insert()  | O(n)           | O(n)  |
| LinkedList   | search()  | O(n)           | O(n)  |
| LinkedList   | remove()  | O(n)           | O(n)  |
| HashMap      | put()     | O(1)           | O(n)  |
| HashMap      | get()     | O(1)           | O(n)  |
| HashMap      | remove()  | O(1)           | O(n)  |


Amortized O(1) insertion means that although some insertions take O(n) time when the array needs to resize, most insertions take O(1), so the average cost per insertion over many insertions is O(1).
---

## Design Decisions

* **DynamicArray** was chosen over a fixed-size array because storage requirements are unknown at runtime.
* **Singly LinkedList** was chosen over a doubly linked list to reduce memory overhead and simplify implementation.
* **HashMap** was selected because Redis Lite requires fast key-based retrieval.
* **Separate Chaining** is used for collision handling because it integrates naturally with linked lists and simplifies insertion and deletion.
* **Rehashing** is performed when the load factor becomes too high, maintaining efficient average-case performance.

---

## Example Usage

```text
> SET name Uvek
OK

> GET name
Uvek

> EXISTS name
true

> DEL name
OK

> SIZE
0
```
