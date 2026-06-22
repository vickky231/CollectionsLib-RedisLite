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

### Generic Programming Using Templates

Templates allow the same implementation to work with different data types.
Generic Dynamic Array
template<typename T>
class DynamicArray
{
    T* data;
};

Examples:
DynamicArray<int> numbers;
DynamicArray<string> names;
 template<typename T>
•	T is a placeholder type. 
•	It can represent any data type such as int, float, string, etc. 
•	The compiler replaces T with the actual type when an object is created. 
 class DynamicArray
•	Defines a generic class named DynamicArray. 
 T* data;
•	data is a pointer to type T. 
•	It can point to an array of any data type specified when creating the object.

________________________________________
Generic Linked List
template<typename T>
struct Node
{
    T data;
    Node<T>* next;
};
Examples:
LinkedList<int> marks;
LinkedList<string> cities;
template<typename T>
•	T is a placeholder for any data type. 
•	It allows the node to store different types of data. 
 T data;
•	Stores the actual value in the node. 
•	The type depends on what T is replaced with. 
Node<T>* next;
•	A pointer that stores the address of the next node. 
•	This creates the link between nodes in the linked list. 
________________________________________
Generic HashNode
template<typename K, typename V>
struct HashNode
{
    K key;
    V value;
    HashNode<K,V>* next;
};
 template<typename K, typename V>
•	K represents the Key type. 
•	V represents the Value type. 
•	These are placeholders that will be replaced with actual data types when the node is created. 
Example:
HashNode<int, string>
Here:
•	K = int 
•	V = string 
________________________________________
2. K key;
Stores the key of the key-value pair.
Example:
key = 101;
or
key = "Name";
________________________________________
3. V value;
Stores the value associated with the key.
Example:
value = "Vivek";
or
value = 95.5;
________________________________________
4. HashNode<K,V>* next;
A pointer to the next node.
It is used when two different keys get mapped to the same hash table index (collision). The nodes are connected like a linked list.
________________________________________
Example 
HashNode<int, string> student;
The compiler treats it as:
struct HashNode
{
    int key;
    string value;
    HashNode* next;
};
Data stored:
Key   = 101
Value = "Vivek"

________________________________________
Generic HashMap
template<typename K, typename V>
class HashMap
{
    HashNode<K,V>** buckets;
};
Examples:
HashMap<string,string> store;
HashMap<int,string> students;

HashNode<K,V>** bucket => This is a pointer to an array of pointers to HashNode objects.
buckets
   |
   v
+------+------+------+------+
|  *   |  *   | NULL |  *   |
+------+------+------+------+
   |          |             |
   v           v             v
 Node   Node      Node
•	Each position in the array is called a bucket. 
•	Each bucket stores the address of the first HashNode. 
•	If multiple items hash to the same bucket, they are linked together using the next pointer.

Redis Lite Instance
HashMap<string,string> store;

### Collision Handling and Rehashing

Separate Chaining
hash("name") % 8 = 1
hash("city") % 8 = 1
Bucket[1]

[name]
   |
   v
[city]

Advantages
•	Prevents overwriting 
•	Simple implementation 
•	Easy insertion and deletion 
________________________________________
Rehashing
Load Factor:
Load Factor = Elements / Buckets
Threshold:
0.75

The 0.75 load factor means the HashMap is allowed to fill up to 75% before it increases its size. This helps keep operations fast and avoids too many collisions.

When exceeded:
1.	Double bucket count. 
2.	Create new bucket table. 
3.	Recompute hash values. 
4.	Reinsert all elements. 
5.	Delete old buckets. 
Purpose:
•	Reduce collisions 
•	Maintain O(1) average performance 


### Redis Lite Design and Memory Management
Supported Commands
SET key value
GET key
DEL key
EXISTS key
COUNT
CLEAR
EXIT

Command Flow
SET key value
      |
      v
Hash Function
      |
      v
Bucket Selection
      |
      v
  Insertion

Memory Management

Dynamic Array
new[]
delete[]
Linked List
new Node
delete Node

HashMap
•	Deletes collision chains. 
•	Deletes bucket array. 
•	Frees old buckets during rehashing.
 
### Rule of Three 

The Rule of Three is a principle in C++ that applies to classes that manage resources such as dynamic memory.

It states:
If a class needs a Destructor, it will usually also need a Copy Constructor and a Copy Assignment Operator.

These three functions work together to ensure that objects manage memory safely.

Why is the Rule of Three Needed?
When an object contains dynamically allocated memory, copying the object incorrectly can cause multiple objects to point to the same memory location.

This can lead to problems such as:
•	Double deletion of memory 
•	Memory corruption 
•	Program crashes 
•	Unexpected behavior 

The Rule of Three prevents these issues by ensuring that every object properly manages its own copy of the data.
________________________________________
The Three Components
1. Destructor
The destructor is responsible for cleaning up resources when an object is destroyed.
Its purpose is to:
•	Free allocated memory 
•	Prevent memory leaks 
•	Release resources owned by the object 
Think of it as a cleanup worker that removes everything an object was using before the object disappears.
________________________________________
2. Copy Constructor
The copy constructor is used when a new object is created from an existing object.
Its purpose is to:
•	Create a separate copy of the original object's data 
•	Ensure the new object has its own resources 
•	Prevent two objects from sharing the same memory unintentionally 
Think of it as making a complete photocopy instead of giving two people the same original document.
________________________________________
3. Copy Assignment Operator
The copy assignment operator is used when one existing object is assigned to another existing object.
Its purpose is to:
•	Safely replace the current data of an object 
•	Avoid memory leaks 
•	Ensure proper copying of resources 
Think of it as replacing the contents of one notebook with a copy of another notebook while first removing the old contents.
________________________________________
How the Rule of Three is Used in Redis-Lite
In a Redis-Lite project, data structures such as:
•	Dynamic Arrays 
•	Linked Lists 
•	Hash Tables 
•	Hash Nodes 
often use dynamic memory to store data.

For example, a hash table stores buckets, and each bucket may contain linked nodes holding key-value pairs. Since memory is allocated dynamically, simply copying the object can cause multiple objects to share the same memory.
_____________________

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
### Conclusion

Redis Lite combines Dynamic Array, Linked List, and HashMap to build a scalable in-memory key-value database. Dynamic Array provides bucket storage, Linked List handles collisions, and HashMap enables fast key-based access. The project demonstrates generic programming, manual memory management, hashing, collision resolution, and system design principles.


