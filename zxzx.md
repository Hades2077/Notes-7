
Experiment 5: Library Database & Indexing
The Goal: Create 4 collections with validation, insert 10+ docs, create 4 types of indexes, and test performance using explain(). 

JavaScript
use LibraryDB

// 1. CREATE COLLECTIONS WITH VALIDATION
db.createCollection("books", { validator: { $jsonSchema: { bsonType: "object", required: ["isbn", "title", "authorId", "publishedYear", "copiesAvailable"], properties: { isbn: { bsonType: "string" }, title: { bsonType: "string" }, authorId: { bsonType: "int" }, publishedYear: { bsonType: "int", minimum: 1500, maximum: 2025 }, copiesAvailable: { bsonType: "int", minimum: 0 } } } } })

db.createCollection("members", { validator: { $jsonSchema: { bsonType: "object", required: ["memberId", "name", "age", "membershipType"], properties: { memberId: { bsonType: "int" }, name: { bsonType: "string" }, age: { bsonType: "int", minimum: 5, maximum: 100 }, membershipType: { enum: ["Standard", "Premium", "Student"] } } } } })

db.createCollection("borrowRecords", { validator: { $jsonSchema: { bsonType: "object", required: ["recordId", "memberId", "isbn", "borrowDate", "returnDate"], properties: { recordId: { bsonType: "int" }, memberId: { bsonType: "int" }, isbn: { bsonType: "string" }, borrowDate: { bsonType: "date" }, returnDate: { bsonType: "date" } } } } })

db.createCollection("authors", { validator: { $jsonSchema: { bsonType: "object", required: ["authorId", "name", "country"], properties: { authorId: { bsonType: "int" }, name: { bsonType: "string" }, country: { bsonType: "string" } } } } })

// 2. INSERT 10 DOCUMENTS (Using a loop to save you time!)
for(let i=1; i<=10; i++) {
    db.books.insertOne({ isbn: "B00"+i, title: "Book Title "+i, authorId: i, publishedYear: 2020, copiesAvailable: 5 });
    db.members.insertOne({ memberId: i, name: "Member "+i, age: 20+i, membershipType: "Standard" });
    db.borrowRecords.insertOne({ recordId: i, memberId: i, isbn: "B00"+i, borrowDate: new Date("2025-01-01"), returnDate: new Date("2024-01-01") });
    db.authors.insertOne({ authorId: i, name: "Author "+i, country: "India" });
}

// 3. CREATE INDEXES
db.books.createIndex({ isbn: 1 })                                // Single-field
db.borrowRecords.createIndex({ memberId: 1, borrowDate: -1 })    // Compound
db.books.createIndex({ title: "text" })                          // Text
db.members.createIndex({ memberId: 1 }, { unique: true })        // Unique

// 4. PERFORMANCE TEST (Explain)
db.books.dropIndex({ isbn: 1 })                                  // Test Without Index
db.books.find({ isbn: "B005" }).explain("executionStats")
db.books.createIndex({ isbn: 1 })                                // Test With Index
db.books.find({ isbn: "B005" }).explain("executionStats")

// 5. ADDITIONAL QUERIES
db.books.createIndex({ authorId: 1 })
db.books.find({ authorId: 5 }).explain("executionStats")                 // Find by Author
db.books.find({ $text: { $search: "Title" } }).explain("executionStats") // Text Search
db.borrowRecords.find({ returnDate: { $lt: new Date() } })               // Find Overdue
db.borrowRecords.find({ memberId: 1 }).sort({ borrowDate: -1 })          // Borrowing History
________________________________________

Experiment 6: Sharding Performance
The Goal: Prove that a database distributed across multiple shards (servers) runs faster than a standard database by comparing execution stats on 1,000,000 records. 

Exam Hack: The terminal setup commands just need to be executed in order. The Javascript for generating 1 million records is already a loop, so you just need to remember the core logic.

Part 1: Terminal Setup (Run these outside mongo)

Bash
# 1. Setup Config Server
mkdir -p /data/configdb
mongod --configsvr --replSet configReplSet --dbpath /data/configdb --port 27019
# In a new terminal, connect via: mongosh --port 27019
rs.initiate({ _id: "configReplSet", configsvr: true, members: [ { _id: 0, host: "localhost:27019" } ] })

# 2. Setup 2 Shard Servers
mkdir -p /data/shard1 /data/shard2
mongod --shardsvr --replSet shard1Repl --dbpath /data/shard1 --port 27018
mongod --shardsvr --replSet shard2Repl --dbpath /data/shard2 --port 27020
# Connect to port 27018 and run: rs.initiate({ _id: "shard1Repl", members: [ { _id: 0, host: "localhost:27018" } ] })
# Connect to port 27020 and run: rs.initiate({ _id: "shard2Repl", members: [ { _id: 0, host: "localhost:27020" } ] })

# 3. Start Router and Add Shards
mongos --configdb configReplSet/localhost:27019 --port 27017
# Connect to port 27017 (mongosh --port 27017) and run:
sh.addShard("shard1Repl/localhost:27018")
sh.addShard("shard2Repl/localhost:27020")
Part 2: Database Operations (Run inside mongosh on port 27017)

JavaScript
use perfTestDB

// Generate Unsharded Data
for (let i = 0; i < 1000000; i++) {
  db.movies_unsharded.insertOne({ title: "Movie " + i, year: 2000 + (i % 20), rating: 5 })
}

// Generate Sharded Data
sh.enableSharding("perfTestDB")
sh.shardCollection("perfTestDB.movies_sharded", { year: 1 })
for (let i = 0; i < 1000000; i++) {
  db.movies_sharded.insertOne({ title: "Movie " + i, year: 2000 + (i % 20), rating: 5 })
}

// Compare Performance (Look at executionTimeMillis in output)
db.movies_unsharded.find({ year: 2010 }).explain("executionStats")
db.movies_sharded.find({ year: 2010 }).explain("executionStats")
________________________________________
Experiment 7: Python Application with MongoDB
The Goal: Use pymongo and tkinter to create a basic GUI app that inserts and views student records. 

Exam Hack: I have combined the tkinter .grid() placement onto the same lines as the widget creations. This cuts the physical length of the code you have to memorize in half without changing how the app looks or functions.

Python
import tkinter as tk
from pymongo import MongoClient

# Database Connection
collection = MongoClient("mongodb://localhost:27017/")["collegeDB"]["students"]

# Functions
def insert_student():
    collection.insert_one({
        "name": e_name.get(), "roll": e_roll.get(), 
        "branch": e_branch.get(), "cgpa": e_cgpa.get()
    })
    for e in (e_name, e_roll, e_branch, e_cgpa): e.delete(0, tk.END)

def display_students():
    result_text.delete("1.0", tk.END)
    for s in collection.find():
        result_text.insert(tk.END, f"{s['name']} | {s['roll']} | {s['branch']} | {s['cgpa']}\n")

# GUI Layout
app = tk.Tk()
app.title("Student DB")

tk.Label(app, text="Name:").grid(row=0, column=0)
e_name = tk.Entry(app); e_name.grid(row=0, column=1)

tk.Label(app, text="Roll No:").grid(row=1, column=0)
e_roll = tk.Entry(app); e_roll.grid(row=1, column=1)

tk.Label(app, text="Branch:").grid(row=2, column=0)
e_branch = tk.Entry(app); e_branch.grid(row=2, column=1)

tk.Label(app, text="CGPA:").grid(row=3, column=0)
e_cgpa = tk.Entry(app); e_cgpa.grid(row=3, column=1)

tk.Button(app, text="Insert", command=insert_student).grid(row=4, column=0, columnspan=2)
tk.Button(app, text="Show All", command=display_students).grid(row=5, column=0, columnspan=2)

result_text = tk.Text(app, width=40, height=10); result_text.grid(row=6, column=0, columnspan=2)
app.mainloop()
________________________________________
 
