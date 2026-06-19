# DataForge & Redis Lite – Design Proposal

## Project Overview

DataForge is a custom C++ collections library consisting of three core data structures: **DynamicArray**, **LinkedList**, and **HashMap**.
These structures are used to build **Redis Lite**, a simplified in-memory key-value database that supports efficient storage, retrieval, and deletion of data.

---

## Public API

### DynamicArray

```cpp
void append(int value);
void insert(int index, int value);
void remove(int index);
int get(int index) const;
void set(int index, int value);
int size() const;
bool isEmpty() const;
```

Provides dynamically resizable contiguous storage with efficient random access.

### LinkedList

```cpp
void insertFront(string key, string value);
void insert(string key, string value);
bool remove(string key);
string search(string key) const;
bool contains(string key) const;
int size() const;
```

Stores key-value pairs and is used for collision handling within HashMap buckets.

### HashMap

```cpp
void put(string key, string value);
string get(string key) const;
bool remove(string key);
bool containsKey(string key) const;
int size() const;
void clear();
```

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

### Architecture

```text
User
  |
  v
Redis Lite CLI
  |
  v
HashMap
  |
  v
DynamicArray (Buckets)
  |
  v
LinkedList (Collision Chains)
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
