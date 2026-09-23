# NoSQL Lab — Experiment 4

## Patient Database, Schema Validation and Indexing using MongoDB

### Aim

Create a patient database containing five collections: `patients`, `doctors`, `medicines`, `prescriptions`, and `sales`. Apply schema validation, insert sample data, create different types of indexes, execute queries using the indexes, verify index usage, and compare query performance with and without indexes.

### Indexes to Demonstrate

1. Single Field Index
2. Compound Index
3. Text Index
4. Partial Index
5. Unique Index

---

# a. Create Database and Collections with Validation

## 1. Switch to Database

```javascript
use patientDB
```

---

## 2. Patients Collection

Age is restricted to **0–120**.

```javascript
db.createCollection("patients", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["patientId", "name", "age", "gender", "phone"],
      properties: {
        patientId: { bsonType: "int" },
        name: { bsonType: "string" },
        age: { bsonType: "int", minimum: 0, maximum: 120 },
        gender: { bsonType: "string" },
        phone: { bsonType: "string" }
      }
    }
  }
})
```

---

## 3. Doctors Collection

```javascript
db.createCollection("doctors", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["doctorId", "name", "specialization"],
      properties: {
        doctorId: { bsonType: "int" },
        name: { bsonType: "string" },
        specialization: { bsonType: "string" }
      }
    }
  }
})
```

---

## 4. Medicines Collection

```javascript
db.createCollection("medicines", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["medicineId", "medicineName", "price", "stock"],
      properties: {
        medicineId: { bsonType: "int" },
        medicineName: { bsonType: "string" },
        price: { bsonType: "double", minimum: 0 },
        stock: { bsonType: "int", minimum: 0 }
      }
    }
  }
})
```

---

## 5. Prescriptions Collection

Dates are stored as MongoDB `Date` values using `ISODate()`.

```javascript
db.createCollection("prescriptions", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["prescriptionId", "patientId", "doctorId", "medicineId", "date"],
      properties: {
        prescriptionId: { bsonType: "int" },
        patientId: { bsonType: "int" },
        doctorId: { bsonType: "int" },
        medicineId: { bsonType: "int" },
        date: { bsonType: "date" }
      }
    }
  }
})
```

---

## 6. Sales Collection

```javascript
db.createCollection("sales", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["saleId", "patientId", "medicineId", "quantity", "amount", "saleDate"],
      properties: {
        saleId: { bsonType: "int" },
        patientId: { bsonType: "int" },
        medicineId: { bsonType: "int" },
        quantity: { bsonType: "int", minimum: 1 },
        amount: { bsonType: "double", minimum: 0 },
        saleDate: { bsonType: "date" }
      }
    }
  }
})
```

---

# b. Insert Sample Data

## 1. Patients — 10 Documents

```javascript
db.patients.insertMany([
  { patientId: 1, name: "Arjun Kumar", age: 25, gender: "Male", phone: "9876543210" },
  { patientId: 2, name: "Priya Sharma", age: 32, gender: "Female", phone: "9876543211" },
  { patientId: 3, name: "Ravi Kumar", age: 45, gender: "Male", phone: "9876543212" },
  { patientId: 4, name: "Meena Rao", age: 28, gender: "Female", phone: "9876543213" },
  { patientId: 5, name: "Amit Singh", age: 52, gender: "Male", phone: "9876543214" },
  { patientId: 6, name: "Kiran Patel", age: 19, gender: "Female", phone: "9876543215" },
  { patientId: 7, name: "Sita Reddy", age: 40, gender: "Female", phone: "9876543216" },
  { patientId: 8, name: "Raj Verma", age: 35, gender: "Male", phone: "9876543217" },
  { patientId: 9, name: "Anita Das", age: 29, gender: "Female", phone: "9876543218" },
  { patientId: 10, name: "Vikram Rao", age: 60, gender: "Male", phone: "9876543219" }
])
```

---

## 2. Doctors — 10 Documents

```javascript
db.doctors.insertMany([
  { doctorId: 1, name: "Dr. Kumar", specialization: "Cardiology" },
  { doctorId: 2, name: "Dr. Meera", specialization: "Neurology" },
  { doctorId: 3, name: "Dr. Ramesh", specialization: "Dermatology" },
  { doctorId: 4, name: "Dr. Anita", specialization: "Pediatrics" },
  { doctorId: 5, name: "Dr. Vijay", specialization: "Orthopedics" },
  { doctorId: 6, name: "Dr. Sunita", specialization: "Gynecology" },
  { doctorId: 7, name: "Dr. Karthik", specialization: "ENT" },
  { doctorId: 8, name: "Dr. Rekha", specialization: "Oncology" },
  { doctorId: 9, name: "Dr. Arvind", specialization: "General Medicine" },
  { doctorId: 10, name: "Dr. Nisha", specialization: "Psychiatry" }
])
```

---

## 3. Medicines — 10 Documents

```javascript
db.medicines.insertMany([
  { medicineId: 1, medicineName: "Paracetamol", price: 20.0, stock: 100 },
  { medicineId: 2, medicineName: "Amoxicillin", price: 50.0, stock: 80 },
  { medicineId: 3, medicineName: "Cetirizine", price: 15.0, stock: 120 },
  { medicineId: 4, medicineName: "Ibuprofen", price: 30.0, stock: 60 },
  { medicineId: 5, medicineName: "Azithromycin", price: 75.0, stock: 90 },
  { medicineId: 6, medicineName: "Omeprazole", price: 40.0, stock: 70 },
  { medicineId: 7, medicineName: "Metformin", price: 35.0, stock: 110 },
  { medicineId: 8, medicineName: "Aspirin", price: 25.0, stock: 50 },
  { medicineId: 9, medicineName: "Pantoprazole", price: 45.0, stock: 65 },
  { medicineId: 10, medicineName: "Loratadine", price: 28.0, stock: 95 }
])
```

---

## 4. Prescriptions — 10 Documents

```javascript
db.prescriptions.insertMany([
  { prescriptionId: 1, patientId: 1, doctorId: 1, medicineId: 1, date: ISODate("2026-01-01") },
  { prescriptionId: 2, patientId: 2, doctorId: 2, medicineId: 2, date: ISODate("2026-01-02") },
  { prescriptionId: 3, patientId: 3, doctorId: 3, medicineId: 3, date: ISODate("2026-01-03") },
  { prescriptionId: 4, patientId: 4, doctorId: 4, medicineId: 4, date: ISODate("2026-01-04") },
  { prescriptionId: 5, patientId: 5, doctorId: 5, medicineId: 5, date: ISODate("2026-01-05") },
  { prescriptionId: 6, patientId: 6, doctorId: 6, medicineId: 6, date: ISODate("2026-01-06") },
  { prescriptionId: 7, patientId: 7, doctorId: 7, medicineId: 7, date: ISODate("2026-01-07") },
  { prescriptionId: 8, patientId: 8, doctorId: 8, medicineId: 8, date: ISODate("2026-01-08") },
  { prescriptionId: 9, patientId: 9, doctorId: 9, medicineId: 9, date: ISODate("2026-01-09") },
  { prescriptionId: 10, patientId: 10, doctorId: 10, medicineId: 10, date: ISODate("2026-01-10") }
])
```

---

## 5. Sales — 10 Documents

```javascript
db.sales.insertMany([
  { saleId: 1, patientId: 1, medicineId: 1, quantity: 2, amount: 40.0, saleDate: ISODate("2026-01-11") },
  { saleId: 2, patientId: 2, medicineId: 2, quantity: 3, amount: 150.0, saleDate: ISODate("2026-01-12") },
  { saleId: 3, patientId: 3, medicineId: 3, quantity: 4, amount: 60.0, saleDate: ISODate("2026-01-13") },
  { saleId: 4, patientId: 4, medicineId: 4, quantity: 2, amount: 60.0, saleDate: ISODate("2026-01-14") },
  { saleId: 5, patientId: 5, medicineId: 5, quantity: 2, amount: 150.0, saleDate: ISODate("2026-01-15") },
  { saleId: 6, patientId: 6, medicineId: 6, quantity: 3, amount: 120.0, saleDate: ISODate("2026-01-16") },
  { saleId: 7, patientId: 7, medicineId: 7, quantity: 4, amount: 140.0, saleDate: ISODate("2026-01-17") },
  { saleId: 8, patientId: 8, medicineId: 8, quantity: 2, amount: 50.0, saleDate: ISODate("2026-01-18") },
  { saleId: 9, patientId: 9, medicineId: 9, quantity: 3, amount: 135.0, saleDate: ISODate("2026-01-19") },
  { saleId: 10, patientId: 10, medicineId: 10, quantity: 5, amount: 140.0, saleDate: ISODate("2026-01-20") }
])
```

---

# c. Creating Indexes

MongoDB supports different types of indexes. The following five types are demonstrated.

---

## i. Single Field Index

Create an index on the `age` field of patients.

```javascript
db.patients.createIndex({ age: 1 })
```

Here:

```text
1 = Ascending
-1 = Descending
```

---

## ii. Compound Index

Create an index on both `specialization` and `name` in the doctors collection.

```javascript
db.doctors.createIndex({
  specialization: 1,
  name: 1
})
```

A compound index contains **multiple fields**.

---

## iii. Text Index

Create a text index on the medicine name.

```javascript
db.medicines.createIndex({
  medicineName: "text"
})
```

This allows text searches using `$text`.

---

## iv. Partial Index

Create an index only for medicines whose stock is less than 70.

```javascript
db.medicines.createIndex(
  { stock: 1 },
  { partialFilterExpression: { stock: { $lt: 70 } } }
)
```

This index contains only documents satisfying:

```text
stock < 70
```

---

## v. Unique Index

Create a unique index on `patientId`.

```javascript
db.patients.createIndex(
  { patientId: 1 },
  { unique: true }
)
```

This prevents two patients from having the same `patientId`.

---

# d. Example Queries Using Indexes

## 1. Single Field Index

Find patients whose age is 30.

```javascript
db.patients.find({
  age: 30
})
```

The `age` index can be used for this query.

---

## 2. Compound Index

Find doctors belonging to a particular specialization and name.

```javascript
db.doctors.find({
  specialization: "Cardiology",
  name: "Dr. Kumar"
})
```

The compound index:

```javascript
{ specialization: 1, name: 1 }
```

can support this query.

---

## 3. Text Index

Search for medicines containing the word `Para`.

```javascript
db.medicines.find({
  $text: {
    $search: "Paracetamol"
  }
})
```

The text index on `medicineName` is used for text search.

---

## 4. Partial Index

Find medicines with stock below 70.

```javascript
db.medicines.find({
  stock: { $lt: 70 }
})
```

The partial index is specifically created for documents satisfying this condition.

---

## 5. Unique Index

Search for a patient using their unique ID.

```javascript
db.patients.find({
  patientId: 5
})
```

The unique index on `patientId` can be used to quickly locate the patient.

---

# e. Index Verification

Use `getIndexes()` to display indexes created on a collection.

```javascript
db.patients.getIndexes()
```

For doctors:

```javascript
db.doctors.getIndexes()
```

For medicines:

```javascript
db.medicines.getIndexes()
```

---

## Verify Query Uses an Index

Use `explain("executionStats")`.

### Example

```javascript
db.patients.find({
  age: 30
}).explain("executionStats")
```

Look for:

```text
IXSCAN
```

`IXSCAN` means MongoDB is scanning an **index**.

If MongoDB performs:

```text
COLLSCAN
```

it is scanning the entire collection instead of using an index.

### Important

```text
IXSCAN  → Index Scan
COLLSCAN → Collection Scan
```

---

# f. Performance Comparison

## i. Without Index

To demonstrate a collection scan, use `$natural` to force a collection scan:

```javascript
db.patients.find({
  age: 30
}).hint({ $natural: 1 }).explain("executionStats")
```

Look for:

```text
COLLSCAN
```

---

## ii. With Index

First create the index:

```javascript
db.patients.createIndex({
  age: 1
})
```

Then run:

```javascript
db.patients.find({
  age: 30
}).explain("executionStats")
```

Look for:

```text
IXSCAN
```

---

# Index Summary

| Index Type   | Command                                                             |
| ------------ | ------------------------------------------------------------------- |
| Single Field | `createIndex({age:1})`                                              |
| Compound     | `createIndex({specialization:1,name:1})`                            |
| Text         | `createIndex({medicineName:"text"})`                                |
| Partial      | `createIndex({stock:1},{partialFilterExpression:{stock:{$lt:70}}})` |
| Unique       | `createIndex({patientId:1},{unique:true})`                          |

---

# Important Commands to Remember

```javascript
// Create index
db.collection.createIndex({ field: 1 })

// View indexes
db.collection.getIndexes()

// Explain query
db.collection.find({...}).explain("executionStats")
```

### Performance

```text
Without Index → COLLSCAN
With Index    → IXSCAN
```

### Final Result

The patient database was successfully created with five validated collections. Ten documents were inserted into each collection. Single-field, compound, text, partial, and unique indexes were created and verified using queries and `explain("executionStats")`. Query execution was compared using collection scans and index scans.
