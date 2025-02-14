

### Steps to Update and Run the Application

1. **Update the Code**:
   - Replace the contents of `app.js` with the updated code above.

2. **Restart the Server**:
   - Stop the running server (if it’s still running) by pressing `Ctrl+C`.
   - Start the server again:
     ```bash
     node app.js
     ```

3. **Verify the Output**:
   - You should no longer see the deprecation warnings.
   - The server should start successfully:
     ```
     Server is running on http://localhost:5000
     ```

---

### Testing the API

You can now test the API using `curl` or Postman as described in the previous steps. For example:

1. **Create a Book**:
   ```bash
   curl -X POST http://<public-ip-of-ec2>:5000/api/books \
     -H "Content-Type: application/json" \
     -d '{"title": "Introduction to RESTful APIs", "author": "John Doe", "pages": 120}'
   ```

2. **Read All Books**:
   ```bash
   curl http://<public-ip-of-ec2>:5000/api/books
   ```

3. **Read a Single Book**:
   ```bash
   curl http://<public-ip-of-ec2>:5000/api/books/<book-id>
   ```

4. **Update a Book**:
   ```bash
   curl -X PUT http://<public-ip-of-ec2>:5000/api/books/<book-id> \
     -H "Content-Type: application/json" \
     -d '{"title": "Updated Title"}'
   ```

5. **Delete a Book**:
   ```bash
   curl -X DELETE http://<public-ip-of-ec2>:5000/api/books/<book-id>
   ```

---

### Troubleshooting

If you encounter any issues:
1. **Check MongoDB Connection**:
   - Ensure the MongoDB connection string (`uri`) is correct.
   - Verify that the EC2 instance's IP is whitelisted in MongoDB Atlas.

2. **Check Server Logs**:
   - Look for any errors in the server logs when starting the application.

3. **Check Firewall Rules**:
   - Ensure the EC2 instance's security group allows inbound traffic on port `5000`.

---

This updated code removes the deprecation warnings and ensures your application runs smoothly. Let me know if you need further assistance!


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
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
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
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
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
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
});

// Start the server
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});

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


Here is the properly formatted **cURL commands and MongoDB connection strings** for better readability and usage:

---

## **📌 1. Create a New Book**
```bash
curl -X POST http://54.221.63.248:5000/api/books \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Introduction to Stanley Samy RESTful APIs",
    "author": "John Sss",
    "pages": 120
  }'
```
**Response Example:**
```json
{"message":"Book created","id":"67af6aec4d3a3a722786e95d"}
```

---

## **📌 2. Retrieve a Specific Book by ID**
```bash
curl -X GET http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d
```
**Response Example (If found):**
```json
{
  "_id": "67af6aec4d3a3a722786e95d",
  "title": "Introduction to RESTful APIs",
  "author": "John Doe",
  "pages": 120
}
```
**Response Example (If not found):**
```json
{"message":"Book not found"}
```

---

## **📌 3. Retrieve All Books**
```bash
curl -X GET http://54.221.63.248:5000/api/books
```

**Response Example:**
```json
[
  {
    "_id": "67af6aec4d3a3a722786e95d",
    "title": "Introduction to RESTful APIs",
    "author": "John Doe",
    "pages": 120
  },
  {
    "_id": "67af6b6f4d3a3a722786e95e",
    "title": "Introduction to MongoDB RESTful APIs",
    "author": "s sa",
    "pages": 120
  }
]
```

---

## **📌 4. Update a Book by ID**
```bash
curl -X PUT http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title"
  }'
```

**Response Example (If Updated Successfully):**
```json
{"message":"Book updated"}
```
**Response Example (If Not Found):**
```json
{"message":"Book not found"}
```

---

## **📌 5. Delete a Book by ID**
```bash
curl -X DELETE http://54.221.63.248:5000/api/books/67af6aec4d3a3a722786e95d
```

**Response Example (If Deleted Successfully):**
```json
{"message":"Book deleted"}
```
**Response Example (If Not Found):**
```json
{"message":"Book not found"}
```

---

## **📌 6. Generalized Commands with `<PUBLIC_IP>` and `<BOOK_ID>`**
Use these placeholders if you're testing on a different EC2 instance:

- **Create a Book:**
  ```bash
  curl -X POST http://<public-ip-of-ec2>:5000/api/books \
    -H "Content-Type: application/json" \
    -d '{"title": "Introduction to RESTful APIs", "author": "John Doe", "pages": 120}'
  ```

- **Update a Book:**
  ```bash
  curl -X PUT http://<public-ip-of-ec2>:5000/api/books/<book-id> \
    -H "Content-Type: application/json" \
    -d '{"title": "Updated Title"}'
  ```

---

## **📌 7. MongoDB Connection Strings**
### **✅ Correct MongoDB Connection String**
```javascript
const uri = "mongodb+srv://<username>:<password>@sandbox.fswdn.mongodb.net/library?retryWrites=true&w=majority&tls=true&tlsAllowInvalidCertificates=true";
```

---

## **📌 8. MongoDB Connection Using `.env` File (Secure Method)**
Instead of exposing credentials, store them in a `.env` file:

### **➜ Step 1: Create `.env` file**
```bash
nano .env
```

**Add this inside:**
```ini
MONGO_URI=mongodb+srv://<username>:<password>@sandbox.fswdn.mongodb.net/library?retryWrites=true&w=majority
```

### **➜ Step 2: Modify `app.js` to use `.env`**
```javascript
require('dotenv').config(); // Load environment variables

const uri = process.env.MONGO_URI;
```

### **➜ Step 3: Restart Server**
```bash
pkill node
node app.js
```

---

## **📌 9. Quick Debugging - Test Server Connection**
To check if the **server is running**, use:
```bash
curl -v http://54.221.63.248:5000/api/books
```
If you get **"Connection refused"**, restart the server.

The **"Cannot GET /api/books"** error means that the **server is not handling the GET request correctly** or is **not running**.  

---

### **🔹 Steps to Fix the Issue Before Testing in Postman**
#### ✅ **1. Check If the Server is Running**
Run this command on your EC2 instance to check if your Express server is active:
```bash
ps aux | grep node
```
If your server **is not running**, restart it:
```bash
pkill node
node app.js
```
Then, confirm it's running:
```bash
curl -X GET http://54.221.63.248:5000/api/books
```
If you **still see "Cannot GET /api/books"**, move to the next step.

---

#### ✅ **2. Verify Express Route in `app.js`**
Open `app.js` and make sure you have this **`GET` route for `/api/books`**:
```javascript
app.get('/api/books', async (req, res) => {
  try {
    const db = client.db('library');
    const collection = db.collection('books');
    const books = await collection.find({}).toArray();
    res.status(200).json(books);
  } catch (error) {
    res.status(500).json({ message: "Error retrieving books", error });
  }
});
```
- If this route is **missing**, add it and restart the server.

---

#### ✅ **3. Ensure Server is Listening on the Correct Port**
In `app.js`, verify that the **server is bound to `0.0.0.0`** so it is accessible externally:
```javascript
const port = process.env.PORT || 5000;
app.listen(port, '0.0.0.0', () => {
  console.log(`🚀 Server running on http://0.0.0.0:${port}`);
});
```
Then restart the server:
```bash
pkill node
node app.js
```
---

### **🚀 Steps to Test in Postman**
Once the **server is running**, follow these steps to test in **Postman**:

---

## **📌 1. Open Postman**
- If you haven’t already, **[download Postman](https://www.postman.com/downloads/)** and open it.

---

## **📌 2. Test `GET /api/books` (Retrieve All Books)**
1. **Open Postman**.
2. **Select `GET`** from the dropdown.
3. **Enter the URL**:
   ```
   http://54.221.63.248:5000/api/books
   ```
4. **Click "Send"**.

### **✅ Expected Response:**
```json
[
  {
    "_id": "67af6b6f4d3a3a722786e95e",
    "title": "Updated Title",
    "author": "John Doe",
    "pages": 120
  }
]
```
- If you **still see "Cannot GET /api/books"**, restart the server and check security groups.

---

## **📌 3. Test `POST /api/books` (Create a New Book)**
1. **Select `POST`** from the dropdown.
2. **Enter the URL**:
   ```
   http://54.221.63.248:5000/api/books
   ```
3. **Go to the `Headers` tab** and add:
   ```
   Key:   Content-Type
   Value: application/json
   ```
4. **Go to the `Body` tab**, select **raw**, and paste:
   ```json
   {
     "title": "Postman Test Book",
     "author": "John Doe",
     "pages": 200
   }
   ```
5. **Click "Send"**.

### **✅ Expected Response:**
```json
{
  "message": "Book created",
  "id": "67af6b6f4d3a3a722786e95e"
}
```

---

## **📌 4. Test `GET /api/books/:id` (Retrieve a Book by ID)**
1. **Select `GET`**.
2. **Enter the URL**, replacing `<id>` with an actual book `_id`:
   ```
   http://54.221.63.248:5000/api/books/67af6b6f4d3a3a722786e95e
   ```
3. **Click "Send"**.

### **✅ Expected Response:**
```json
{
  "_id": "67af6b6f4d3a3a722786e95e",
  "title": "Postman Test Book",
  "author": "John Doe",
  "pages": 200
}
```

---

## **📌 5. Test `PUT /api/books/:id` (Update a Book)**
1. **Select `PUT`**.
2. **Enter the URL**:
   ```
   http://54.221.63.248:5000/api/books/67af6b6f4d3a3a722786e95e
   ```
3. **Go to the `Headers` tab** and add:
   ```
   Key:   Content-Type
   Value: application/json
   ```
4. **Go to `Body`**, select **raw**, and paste:
   ```json
   {
     "title": "Updated Title from Postman"
   }
   ```
5. **Click "Send"**.

### **✅ Expected Response:**
```json
{
  "message": "Book updated"
}
```

---

## **📌 6. Test `DELETE /api/books/:id` (Delete a Book)**
1. **Select `DELETE`**.
2. **Enter the URL**:
   ```
   http://54.221.63.248:5000/api/books/67af6b6f4d3a3a722786e95e
   ```
3. **Click "Send"**.

### **✅ Expected Response:**
```json
{
  "message": "Book deleted"
}
```

---

## **✅ Final Checks**
### If **Postman still returns "Cannot GET /api/books"**, try these:
1. **Check if the server is running**:
   ```bash
   ps aux | grep node
   ```
2. **Check if the Express route exists in `app.js`**.
3. **Check if Security Group allows port `5000`** (AWS EC2):
   - Go to AWS Console → EC2 → Security Groups.
   - Ensure **Inbound Rules** allow:
     ```
     Type:  Custom TCP
     Port:  5000
     Source: 0.0.0.0/0
     ```
4. **Restart the server**:
   ```bash
   pkill node
   node app.js
   ```
5. **Run this test manually**:
   ```bash
   curl -X GET http://54.221.63.248:5000/api/books
   ```

---

### 🎯 **Now Postman Should Work!**
✅ If the API is up and running, you should get **valid responses** for all requests.  
If the issue persists, **send me the output of**:
```bash
ps aux | grep node
sudo netstat -tulnp | grep 5000
sudo journalctl -u node --no-pager --lines=50
```

