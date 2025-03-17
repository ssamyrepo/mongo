
### 1. **Introduction to MongoDB and NoSQL**
   - **NoSQL Databases**: Designed to handle unstructured or semi-structured data, unlike traditional relational databases (RDBMS) that use tables with fixed schemas.
   - **MongoDB**: A document-oriented NoSQL database that stores data in JSON-like documents. It is highly scalable, flexible, and supports dynamic schemas.
   - **Advantages of NoSQL**: Better performance, horizontal scalability, and no strict schema design.

### 2. **MongoDB Installation**
   - **Linux Installation**: Steps to install MongoDB Community Edition on Linux, including creating a repository file, installing the package, and starting the MongoDB service.
   - **Windows Installation**: Steps to install MongoDB on Windows, including downloading the installer, configuring the service, and setting up environment variables.

### 3. **Database and Collection Operations**
   - **Creating and Dropping Databases**: Using the `use` command to create a database and `db.dropDatabase()` to drop it.
   - **Collections**: Analogous to tables in RDBMS. Collections can store documents with varying structures.
   - **Capped Collections**: Fixed-size collections that overwrite old data when the size limit is reached.

### 4. **CRUD Operations**
   - **Create**: Inserting documents into a collection using `insertOne()` or `insertMany()`.
   - **Read**: Querying documents using `find()` with optional conditions.
   - **Update**: Updating documents using `updateOne()` or `updateMany()`.
   - **Delete**: Deleting documents using `deleteOne()` or `deleteMany()`.

### 5. **Importing and Exporting Data**
   - **Importing Data**: Using the `mongoimport` utility to import JSON or CSV data into a MongoDB collection.
   - **Exporting Data**: Using the `mongoexport` utility to export data from a collection to a JSON or CSV file.

### 6. **Backup and Restore**
   - **mongodump**: A utility to create binary backups of MongoDB databases or collections.
   - **mongorestore**: A utility to restore data from backups created by `mongodump`.

### 7. **Replication**
   - **Replica Sets**: Configuring a replica set with a primary node and secondary nodes for high availability and data redundancy.
   - **Adding Nodes**: Adding secondary nodes to a replica set and ensuring data replication.

### 8. **Storage Engines**
   - **WiredTiger**: The default storage engine in MongoDB, offering features like document-level concurrency, compression, and checkpointing.
   - **In-Memory Storage Engine**: Available in MongoDB Enterprise Edition, storing all data in memory for faster access.
   - **MMAPv1**: Deprecated storage engine used in versions prior to MongoDB 3.2.

### 9. **Journaling**
   - **Journal Files**: Binary transaction logs used for recovery in case of a crash. Journaling ensures data durability by writing transactions to journal files before committing them to data files.
   - **Checkpointing**: WiredTiger creates checkpoints every 60 seconds by default, ensuring data consistency on disk.

### 10. **Remote Access**
   - **Enabling Remote Access**: Modifying the MongoDB configuration file (`mongod.conf`) to allow remote connections by binding the server to a specific IP address.

### 11. **User Management**
   - **Creating Admin and Application Users**: Creating users with specific roles and permissions, enabling authentication, and logging in with user credentials.

### 12. **JSON and BSON**
   - **JSON**: JavaScript Object Notation, a human-readable data format used in MongoDB for document storage.
   - **BSON**: Binary JSON, a binary-encoded format used internally by MongoDB for efficient storage and retrieval.

### 13. **MongoDB Tools**
   - **MongoDB Compass**: A GUI tool for MongoDB administration.
   - **NoSQL Booster**: Another GUI tool for MongoDB, providing a user-friendly interface for database management.

### 14. **Performance and Scalability**
   - **Indexing**: Improving query performance by creating indexes on fields.
   - **Sharding**: Distributing data across multiple servers to handle large datasets and improve scalability.

### 15. **MongoDB Atlas**
   - **Cloud-Based MongoDB**: Using MongoDB Atlas, a fully managed cloud database service, for deploying and managing MongoDB in the cloud.

### 16. **MongoDB vs SQL**
   - **Differences**: MongoDB is schema-less, supports horizontal scaling, and is ideal for unstructured data, while SQL databases are schema-based, support vertical scaling, and are ideal for structured data with complex relationships.

### 17. **ACID vs BASE**
   - **ACID**: Atomicity, Consistency, Isolation, Durability (used in SQL databases).
   - **BASE**: Basically Available, Soft state, Eventually consistent (used in NoSQL databases).

### 18. **When to Use MongoDB**
   - **Use Cases**: MongoDB is suitable for applications with rapidly changing requirements, large-scale data, and the need for high availability and scalability.
