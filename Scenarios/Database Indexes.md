# Database Indexes: Hash Indexes, B-Trees, and Abstraction

This document provides a detailed explanation of database indexes, covering the foundational concepts, implementation details of Hash Indexes and B-Tree Indexes, and general indexing abstractions.

---

## 1. Index Abstraction and General Purpose

The main point of using indexes is to achieve **faster reads** on a specific key value.

### Index Necessity and Trade-offs
*   **Without an index**, finding desired rows requires going through *all* database rows, resulting in **O(N) (linear time)** complexity reads. This is considered "super complicated" for large tables.
*   An index effectively **sorts the database table** by the chosen key.
*   This sorting reduces read time complexity from a linear scan to a **logarithmic time scan (O(log N))**.
*   **The Trade-off (The Write Penalty):** Every single write performed on the database table is **significantly slower**. This slowdown is due to the logarithmic time complexity incurred when updating structures like B-trees.
*   In systems design, indexes represent a common trade-off: optimizing for faster reads at the cost of slower writes.

### Index Types (Data Storage Abstraction)

When designing an index, databases must choose how much row data to store directly within the index structure:

*   **Clustered Index:** The **data of the row itself** is stored within the index (e.g., the date and text of a post).
    *   **Benefit:** Provides **faster reads** because the data can be accessed directly once the index is searched.
*   **Address Index (More common in practice):** The index only stores the **address of the corresponding row on disk**.
    *   **Benefit:** Leads to **less data duplication**. This is crucial when a database has multiple indexes, as it prevents the entire row data from being duplicated across every index.
*   **Covered Index (Middle Ground):** Stores a **subset of important fields** alongside the disk address. This balances the benefit of faster reads for specific fields with the goal of storing less data than a clustered index.

---

## 2. Hash Indexes

Hash indexes are based on a hash map and are one specific type of database index.

### Implementation and Performance
*   Hash indexes are typically kept **in memory (RAM)**.
*   They map the key (via a hash function) to the location (address) of the row on disk.
*   **Performance:** All operations (reads and writes) are generally **O(1) (constant time)**. This speed is achieved because RAM allows for random access jumps across the array/memory location quickly.

### Constraints and Downsides
*   **Range Queries are Not Supported:** Hashmaps are designed to distribute elements evenly. Since similar keys are not adjacent, checking for rows within a range (e.g., names between A and B) is not feasible. To perform a range query, the system would need to loop through all indexes (O(N)) or check literally every possible string/value between the range boundaries, which is not efficient.
*   **Memory Limit:** All keys must fit in RAM. Since RAM is expensive, this constrains the maximum size of the data set.
*   **Poor Disk Performance:** Hashmaps are inherently bad on disk because the key values are distributed all over the disk, requiring the pointer to jump around the metallic disc frequently. Hash indexes are always kept in memory as a result.

### Durability
*   RAM is not durable; data is lost if the computer shuts down.
*   To achieve durability, hash indexes rely on a **write ahead log (WAL)**.
*   The WAL is a sequentially written log stored on disk, recording every write and update made to the database. Sequential writes are relatively fast on a hard drive.
*   If the computer dies, the hash index can be **repopulated by replaying all the changes** sequentially from the WAL.
*   Using the WAL slows down write operations because disk writes are slower than memory writes.

---

## 3. B-Tree Indexes

The B-Tree is a type of database index kept **entirely on disk**. It is a self-balancing tree structure.

### Structure and Speed
*   The B-tree gives ranges of keys and contains references (pointers) to other locations on disk that must be traversed to find the desired row.
*   B-tree pages (nodes) are typically large (e.g., 256 megabytes).
*   Keeping pages large ensures the tree remains relatively short. This is critical because it means fewer jumps are required around the disk, which optimizes for speed.
*   **Read Performance:** B-trees are considered relatively speedy for reads because they require only a couple of traversals on disk to find the key.

### Advantages
*   **Supports Range Queries:** Keys that are similar (adjacent) to one another are physically next to each other in the child pages. This inherent sorting allows the system to fetch large ranges of data (e.g., keys between E and L) efficiently with fewer disk reads.
*   **Handles Large Data Sets:** Not all keys have to fit in memory. B-trees can support a much larger data set size (limited only by hard drive capacity).
*   The process of splitting nodes ensures the B-tree remains **balanced**.

### Write Process and Node Splitting
The general process for writing (or updating) involves traversing the tree to the leaf node where the key should exist. If the key is updated, it is updated in place. If a new key is added, complexity arises:

1.  **If Space Exists:** If the leaf node has extra space, the key is added while maintaining the internal alphabetical sorted order. The specific node must be written/updated.
2.  **If No Space Exists (Node Splitting):** If the leaf node is full, it must be **split into two nodes**.
3.  The parent node must then be updated to delineate the new key boundaries (e.g., changing the range to A to BE, and BO to C).
4.  If the **parent node also lacks space**, the splitting process must **continue traversing up the tree** until free space is found.
5.  If the **root node** lacks free space, the root itself is split into two, and a **new root node** is created above the original root.

### Durability and Write Speed
*   The B-tree uses a **write ahead log (WAL)** to ensure consistency and prevent corruption if the system crashes mid-update.
*   Before any write or update is made to the B-tree, it is first recorded in the WAL.
*   If the computer turns off, the system can replay the transactions in the WAL to recover the consistent state of the index.
*   **Write Speed Penalty:** B-tree writes are not super fast. They require multiple writes: to the WAL, to the bottom of the tree, and then potentially traversing all the way back up to switch disk references.

---

## 4. Multi-dimensional / Composite Indexes

This structure allows for sorting based on more than one field:

*   A composite index defines **one primary sort order** and then an **internal sort order** on a second field.
*   **Example:** Indexing first on `username` and then internally on `date`. The result is that all posts are sorted by username, and within each username, the posts are sorted by date.
*   This differs from having two separate indexes, which would sort each field independently across the entire database table (e.g., sorting everything chronologically by date regardless of username).
