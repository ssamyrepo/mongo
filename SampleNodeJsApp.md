
### **1. Fix `_id` Handling for ObjectId**
Modify the **`app.js`** file to correctly **convert the `id` string to ObjectId** before querying MongoDB.

#### **Updated `app.js`**
```javascript
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb'); // Import ObjectId
const bodyParser = require('body-parser');

const app = express();
const port = 5000;

// MongoDB connection string
const uri = "mongodb+srv://<username>:<password>@<cluster-url>/<dbname>?retryWrites=true&w=majority";
const client = new MongoClient(uri);

// Middleware to parse JSON
app.use(bodyParser.json());

// Connect to MongoDB
async function connectToDB() {
  try {
    await client.connect();
    console.log("Connected to MongoDB");
  } catch (err) {
    console.error("Error connecting to MongoDB:", err);
  }
}

connectToDB();

// Create a new document
app.post('/api/books', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const result = await collection.insertOne(req.body);
  res.status(201).json({ message: "Book created", id: result.insertedId });
});

// Read all documents
app.get('/api/books', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');
  const books = await collection.find({}).toArray();
  res.status(200).json(books);
});

// Read a single document by ID
app.get('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');

  try {
    const book = await collection.findOne({ _id: new ObjectId(req.params.id) });
    if (book) {
      res.status(200).json(book);
    } else {
      res.status(404).json({ message: "Book not found" });
    }
  } catch (error) {
    res.status(400).json({ message: "Invalid ID format" });
  }
});

// Update a document by ID
app.put('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');

  try {
    const result = await collection.updateOne(
      { _id: new ObjectId(req.params.id) },
      { $set: req.body }
    );
    if (result.modifiedCount > 0) {
      res.status(200).json({ message: "Book updated" });
    } else {
      res.status(404).json({ message: "Book not found" });
    }
  } catch (error) {
    res.status(400).json({ message: "Invalid ID format" });
  }
});

// Delete a document by ID
app.delete('/api/books/:id', async (req, res) => {
  const db = client.db('library');
  const collection = db.collection('books');

  try {
    const result = await collection.deleteOne({ _id: new ObjectId(req.params.id) });
    if (result.deletedCount > 0) {
      res.status(200).json({ message: "Book deleted" });
    } else {
      res.status(404).json({ message: "Book not found" });
    }
  } catch (error) {
    res.status(400).json({ message: "Invalid ID format" });
  }
});

// Start the server
app.listen(port, () => {
  console.log(`Server is running on http://0.0.0.0:${port}`);
});
```

---

### **2. Restart Your Server**
After updating the code, restart the server:
```bash
pkill node
node app.js
```

---

### **3. Test cURL Commands**

#### **Get All Books**
```bash
curl -v http://54.221.63.248:5000/api/books
```

#### **Retrieve a Book by ID**
Replace `<book_id>` with an actual ID from your database:
```bash
curl -v http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d
```

#### **Update a Book by ID**
```bash
curl -v -X PUT http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d \
-H "Content-Type: application/json" \
-d '{"title": "Updated Title", "author": "Updated Author", "pages": 200}'
```

#### **Delete a Book by ID**
```bash
curl -v -X DELETE http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d
```

---

### **4. Explanation of Fixes**
1. **Converted `req.params.id` to `ObjectId(req.params.id)`** in:
   - `findOne()`
   - `updateOne()`
   - `deleteOne()`
   This ensures MongoDB treats the `_id` as an **ObjectId** instead of a **string**.

2. **Handled Invalid ID Errors**:
   - If the provided `_id` is not a **valid 24-character hex string**, the server responds with:
     ```json
     {"message": "Invalid ID format"}
     ```

3. **Made the Server Bind to `0.0.0.0`**:
   - Now, external devices (not just localhost) can access it.

---

### **5. Expected Outcome**
Your API should now correctly retrieve, update, and delete books by `_id`! 🚀 Try it out and let me know if you need any further assistance.
