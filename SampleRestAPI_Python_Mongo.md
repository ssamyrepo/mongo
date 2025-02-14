## Prerequisites

1. **Python**: Ensure Python 3.x is installed on your EC2 instance.
2. **MongoDB Atlas**: Use the same MongoDB Atlas cluster as before.
3. **Flask and PyMongo**: Install the required Python libraries.

---

## Step 1: Install Required Libraries

1. **Install Flask and PyMongo**:
   ```bash
   pip install Flask pymongo
   ```

2. **Verify Installation**:
   ```bash
   python --version
   pip show Flask pymongo
   ```

---

## Step 2: Create the Python REST API

1. **Create a Project Directory**:
   ```bash
   mkdir python-rest-api
   cd python-rest-api
   ```

2. **Create the Application File**:
   - Create a file named `app.py`:
     ```bash
     touch app.py
     ```

3. **Write the Python Code**:
   - Open `app.py` and add the following code:

```python
from flask import Flask, jsonify, request
from pymongo import MongoClient
from bson.objectid import ObjectId
from bson.json_util import dumps

app = Flask(__name__)

# MongoDB connection string
uri = "mongodb+srv://<username>:<password>@<cluster-url>/<dbname>?retryWrites=true&w=majority"
client = MongoClient(uri)
db = client.library  # Use the 'library' database
collection = db.books  # Use the 'books' collection

# Create a new book
@app.route('/api/books', methods=['POST'])
def create_book():
    data = request.get_json()
    if not data:
        return jsonify({"message": "No data provided"}), 400

    result = collection.insert_one(data)
    return jsonify({"message": "Book created", "id": str(result.inserted_id)}), 201

# Get all books
@app.route('/api/books', methods=['GET'])
def get_all_books():
    books = list(collection.find({}))
    return dumps(books), 200

# Get a single book by ID
@app.route('/api/books/<id>', methods=['GET'])
def get_book(id):
    try:
        book = collection.find_one({"_id": ObjectId(id)})
        if book:
            return dumps(book), 200
        else:
            return jsonify({"message": "Book not found"}), 404
    except Exception as e:
        return jsonify({"message": "Invalid ID"}), 400

# Update a book by ID
@app.route('/api/books/<id>', methods=['PUT'])
def update_book(id):
    data = request.get_json()
    if not data:
        return jsonify({"message": "No data provided"}), 400

    try:
        result = collection.update_one({"_id": ObjectId(id)}, {"$set": data})
        if result.modified_count > 0:
            return jsonify({"message": "Book updated"}), 200
        else:
            return jsonify({"message": "Book not found"}), 404
    except Exception as e:
        return jsonify({"message": "Invalid ID"}), 400

# Delete a book by ID
@app.route('/api/books/<id>', methods=['DELETE'])
def delete_book(id):
    try:
        result = collection.delete_one({"_id": ObjectId(id)})
        if result.deleted_count > 0:
            return jsonify({"message": "Book deleted"}), 200
        else:
            return jsonify({"message": "Book not found"}), 404
    except Exception as e:
        return jsonify({"message": "Invalid ID"}), 400

# Start the server
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## Step 3: Run the Application

1. **Start the Server**:
   ```bash
   python app.py
   ```

2. **Verify the Output**:
   - The server should start successfully:
     ```
     * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
     ```

---

## Step 4: Test the API

You can use `curl` or Postman to test the API endpoints.

---

### Example Test Cases

#### 1. **Create a Book**
```bash
curl -X POST http://<public-ip-of-ec2>:5000/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Introduction to RESTful APIs", "author": "John Doe", "pages": 120}'
```

#### 2. **Read All Books**
```bash
curl http://<public-ip-of-ec2>:5000/api/books
```

#### 3. **Read a Single Book**
```bash
curl http://<public-ip-of-ec2>:5000/api/books/<book-id>
```

#### 4. **Update a Book**
```bash
curl -X PUT http://<public-ip-of-ec2>:5000/api/books/<book-id> \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}'
```

#### 5. **Delete a Book**
```bash
curl -X DELETE http://<public-ip-of-ec2>:5000/api/books/<book-id>
```

---

## Step 5: Expected Output

- **Create a Book**:
  ```json
  {
    "message": "Book created",
    "id": "<book-id>"
  }
  ```

- **Read All Books**:
  ```json
  [
    {
      "_id": "<book-id>",
      "title": "Introduction to RESTful APIs",
      "author": "John Doe",
      "pages": 120
    }
  ]
  ```

- **Read a Single Book**:
  ```json
  {
    "_id": "<book-id>",
    "title": "Introduction to RESTful APIs",
    "author": "John Doe",
    "pages": 120
  }
  ```

- **Update a Book**:
  ```json
  {
    "message": "Book updated"
  }
  ```

- **Delete a Book**:
  ```json
  {
    "message": "Book deleted"
  }
  ```

---

## Step 6: Clean Up

1. **Stop the Server**:
   - Press `Ctrl+C` to stop the Flask server.

2. **Terminate the EC2 Instance**:
   - Go to the AWS EC2 console and terminate the instance if no longer needed.

