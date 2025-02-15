### **MongoDB Journaling: Why It Matters**
**Journaling** is a feature in MongoDB that ensures **data durability** and **prevents corruption** in case of unexpected failures (power loss, system crashes, or abrupt shutdowns). It achieves this by **writing changes to a journal file before applying them to the main database**.

---

## **How Journaling Works in MongoDB**
1. **Write Operation Initiated**  
   - When a client issues a `write` operation (`INSERT`, `UPDATE`, `DELETE`), MongoDB does **not** write it directly to the data files (`.wt` files in WiredTiger storage engine).
   
2. **Changes Logged in Journal File (`.wtj`)**  
   - Instead, the operation is **first written to the journal file** (`.wtj`) inside the `journal/` directory.
   - This is a **separate disk location** from the main database.

3. **Data Written to Memory (RAM)**  
   - MongoDB temporarily keeps the changes in memory for better performance.

4. **Commit to Disk (Checkpointing)**  
   - Every **100ms (default)** or **whenever journal reaches a threshold**, MongoDB flushes the data from the journal to the main data files.
   - The changes are now **fully committed and durable**.

---

## **Why Journaling is Important?**
| **Scenario** | **Without Journaling** | **With Journaling Enabled (`storage.journal.enabled: true`)** |
|-------------|-------------------------|--------------------------------|
| **Power Failure** | **Data corruption**; incomplete writes may leave database in an inconsistent state. | MongoDB can **recover from journal logs** to restore last consistent state. |
| **System Crash** | Data loss may occur if data was in memory but not written to disk. | Journal entries ensure that no committed write is lost. |
| **Filesystem Errors** | MongoDB database files may become corrupted. | Automatic **replay of journaled changes** on restart. |
| **Abrupt Shutdown (kill process)** | Requires full **repair operation** (`mongod --repair`). | MongoDB replays **journaled operations automatically**. |

---

## **How to Enable Journaling?**
By default, **journaling is enabled** in the **WiredTiger storage engine**.

### **Check if Journaling is Enabled**
Run the following command in the **MongoDB shell**:
```sh
db.serverStatus().storageEngine
```
Expected Output:
```json
{
  "name": "wiredTiger",
  "supportsCommittedReads": true,
  "persistent": true,
  "journal": { "enabled": true }
}
```

### **Enable Journaling in MongoDB Configuration**
If journaling is disabled, enable it manually in the MongoDB **configuration file (`mongod.conf`)**:

```yaml
storage:
  journal:
    enabled: true
```

Restart MongoDB:
```sh
sudo systemctl restart mongod
```

---

## **How MongoDB Recovers from a Crash?**
1. **MongoDB detects an unclean shutdown** (e.g., system crash).
2. **On startup, it checks journal files (`.wtj`)** for uncommitted operations.
3. **If journal exists, MongoDB replays the operations** to restore the last consistent state.
4. **Database is now recovered**, and normal operations continue.

---

## **Performance Impact of Journaling**
- **Minimal performance overhead (~10% additional disk usage).**
- Recommended to **use SSD-backed EBS volumes** to handle frequent journal writes efficiently.
- Can be tuned by adjusting **journal commit interval (`journalCompressor` setting)**.

---

### **When Should You Disable Journaling?**
- If **MongoDB is running in a temporary/test environment** where data durability isn’t a concern.
- If **running on RAM-disk** where disk writes aren’t needed.
- **Not recommended** for production databases, as it **increases risk of data loss**.

---

## **Final Recommendation**
✅ Always **keep journaling enabled** for **production MongoDB deployments** to prevent **data corruption and improve crash recovery**. 🚀
