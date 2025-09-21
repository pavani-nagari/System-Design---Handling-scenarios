```markdown
# Database Concurrency and Race Conditions

Databases function as **multi-threaded** systems, executing many read and write operations concurrently [Conversation History]. Because the exact order of these operations is not deterministic, the system is susceptible to **race conditions**, which threaten the correctness and consistency of the database [Conversation History].

A transaction's operations are only considered **committed** once all associated rights are successful [Conversation History]. If a transaction fails mid-way, the entire sequence of writes is considered uncommitted [Conversation History].

---

## 1. The Dirty Write Race Condition

A **dirty write** occurs when one thread writes over **uncommitted values** created by another thread [Conversation History].

### Scenario and Inconsistency
If two concurrent threads (T1 and T2) attempt to update the same set of rows simultaneously, and T2's rights reach the database faster than T1's, an inconsistent result can occur [Conversation History].

*   **Example:** T1 attempts to set a purchaser and address, while T2 attempts to set the same purchaser and address [Conversation History]. If T2 wins the `purchaser` write, but T1 wins the `delivery address` write, the result is an inconsistent state where the purchaser and the delivery address do not match [Conversation History].

### Resolution
Dirty writes are typically prevented using **row level locks** [Conversation History].

*   If one thread grabs a lock on a row, no other thread can perform any operation on that row until the lock is released [Conversation History].
*   A risk of this approach is **deadlock**, where two threads need resources held by the other, forcing one transaction to give up its locks [Conversation History].

---

## 2. The Dirty Read Race Condition

A **dirty read** is defined as reading uncommitted data [Conversation History].

### Scenario and Inconsistency
If a reading transaction reads data that is part of a write transaction that has not yet committed, and the write transaction subsequently fails, the reading transaction has seen incorrect data [Conversation History].

*   **Example:** Transaction 1 removes $10 from a bank account and then attempts to add $10 to a second account [Conversation History]. If Transaction 2 checks the balance *after* the initial removal but *before* the transaction is committed, Transaction 2 reads an uncommitted value [Conversation History]. If the second part of Transaction 1 fails, the entire transaction is rolled back, meaning the balance was never truly reduced by $10, yet Transaction 2 believed it was [Conversation History].

### Resolution
The preferred method for preventing dirty reads is **not** by using locks, as locks are slow and would bottleneck fast read operations [Conversation History].

*   The fix is to **store the old value until the commit** [Conversation History]. The database stores the old, committed value alongside the new, uncommitted value, allowing reading threads to access the old, consistent value [Conversation History]. Once the entire sequence of writes is successful, the pointer is switched to the new value [Conversation History].

---

## 3. The Read Skew (Repeatable Read) Race Condition

**Read Skew**, also known as **repeatable read**, is a more complex race condition where a transaction performing a long read operation sees the database in an inconsistent state due to interjecting concurrent write operations. This issue arises even though the read operation is only seeing committed data.

### Scenario and Inconsistency
This problem is best illustrated by maintaining a database invariant (a rule that must always hold), such as the total balance of all accounts adding up to one million dollars.

*   **Long Read:** A user initiates a massive read to calculate the total sum of all accounts.
*   **Interjecting Write:** Midway through the read, a second transaction writes new, committed values (e.g., Chris sends Caitlyn $100,000).
*   **Inconsistent View:** The massive read records Caitlyn's old value ($100K) before the write occurred. When the read reaches Chris, it records Chris's new value ($0).
*   **Result:** Because the read transaction failed to account for the $100K transfer that occurred in the middle of the read, the calculated sum is incorrect, breaking the database invariant during the reading process.

### Resolution: Snapshot Isolation

To solve read skew, the database implements **Snapshot Isolation**, which ensures that the long read sees a **consistent snapshot of the database** taken at a specific time.

#### Implementation Steps:

1.  **Ordered Transactions:** Every write transaction is logged sequentially in the **Write Ahead Log (WAL)**, and a **monotonically increasing sequence number** is assigned to define the order of transactions.
2.  **Data Versioning:** When a row is modified, the old value is **not deleted**. Instead, all previous versions of the row are stored alongside the transaction number that wrote them. These versions are relatively sparse and do not require excessive storage.
3.  **Snapshot Generation:** To generate a snapshot (e.g., at Transaction 15, or T15), the system ignores any writes that occurred *after* T15. For every row, it finds the **most recent value that was committed before T15**.
    *   If a row was only written after T15 (e.g., at T22), that row **does not exist** in the T15 snapshot.

By providing this versioned snapshot, long reads do not risk seeing the database in an inconsistent state.
