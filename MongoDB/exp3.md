# NoSQL Lab — Experiment 3

## Pharmacy Database using MongoDB

### Aim

Create a **pharmacy database** containing `customers`, `medicines`, `suppliers`, `orders`, and `pharmacists` collections. Apply schema validation constraints, insert at least 10 documents into each collection, and perform multi-collection aggregation queries.

### Required Operations

1. Switch to database
2. Create collections with validation
3. Insert 10 documents into each collection
4. Perform aggregation queries:

   * List all orders with customer name and medicine name
   * Total sales per medicine
   * Supplier-wise revenue
   * Medicines below stock threshold
   * Customer order summary

---

# a. Switch to Database

```javascript
use pharmacyDB
```

---

# b. Create Collections with Validation

## 1. Customers

Constraint:

* Age must be **≥ 18**

```javascript
db.createCollection("customers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["customerId", "name", "age", "phone"],
      properties: {
        customerId: { bsonType: "int" },
        name: { bsonType: "string" },
        age: { bsonType: "int", minimum: 18 },
        phone: { bsonType: "string" }
      }
    }
  }
})
```

---

## 2. Medicines

Constraints:

* Expiry date must be a BSON/ISO date
* Stock must be **≥ 0**

```javascript
db.createCollection("medicines", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["medicineId", "medicineName", "supplierId", "price", "stock", "expiryDate"],
      properties: {
        medicineId: { bsonType: "int" },
        medicineName: { bsonType: "string" },
        supplierId: { bsonType: "int" },
        price: { bsonType: "double", minimum: 0 },
        stock: { bsonType: "int", minimum: 0 },
        expiryDate: { bsonType: "date" }
      }
    }
  }
})
```

> `ISODate("YYYY-MM-DD")` creates a valid MongoDB date value.

---

## 3. Suppliers

```javascript
db.createCollection("suppliers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["supplierId", "supplierName", "contact"],
      properties: {
        supplierId: { bsonType: "int" },
        supplierName: { bsonType: "string" },
        contact: { bsonType: "string" }
      }
    }
  }
})
```

---

## 4. Orders

Each order contains:

* Customer
* Medicine
* Quantity
* Price
* Pharmacist
* Order date

```javascript
db.createCollection("orders", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: [
        "orderId",
        "customerId",
        "medicineId",
        "pharmacistId",
        "quantity",
        "totalAmount",
        "orderDate"
      ],
      properties: {
        orderId: { bsonType: "int" },
        customerId: { bsonType: "int" },
        medicineId: { bsonType: "int" },
        pharmacistId: { bsonType: "int" },
        quantity: { bsonType: "int", minimum: 1 },
        totalAmount: { bsonType: "double", minimum: 0 },
        orderDate: { bsonType: "date" }
      }
    }
  }
})
```

---

## 5. Pharmacists

```javascript
db.createCollection("pharmacists", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["pharmacistId", "name", "licenseNo"],
      properties: {
        pharmacistId: { bsonType: "int" },
        name: { bsonType: "string" },
        licenseNo: { bsonType: "string" }
      }
    }
  }
})
```

---

# c. Insert Sample Data

## 1. Customers — 10 Documents

```javascript
db.customers.insertMany([
  { customerId: 1, name: "Arjun", age: 25, phone: "9876543210" },
  { customerId: 2, name: "Priya", age: 30, phone: "9876543211" },
  { customerId: 3, name: "Ravi", age: 42, phone: "9876543212" },
  { customerId: 4, name: "Meena", age: 28, phone: "9876543213" },
  { customerId: 5, name: "Amit", age: 35, phone: "9876543214" },
  { customerId: 6, name: "Kiran", age: 22, phone: "9876543215" },
  { customerId: 7, name: "Sita", age: 45, phone: "9876543216" },
  { customerId: 8, name: "Raj", age: 31, phone: "9876543217" },
  { customerId: 9, name: "Anita", age: 27, phone: "9876543218" },
  { customerId: 10, name: "Vikram", age: 50, phone: "9876543219" }
])
```

---

## 2. Suppliers — 10 Documents

```javascript
db.suppliers.insertMany([
  { supplierId: 101, supplierName: "MediCare Ltd", contact: "9000000001" },
  { supplierId: 102, supplierName: "HealthPlus", contact: "9000000002" },
  { supplierId: 103, supplierName: "PharmaCorp", contact: "9000000003" },
  { supplierId: 104, supplierName: "Wellness Drugs", contact: "9000000004" },
  { supplierId: 105, supplierName: "LifeCare Pharma", contact: "9000000005" },
  { supplierId: 106, supplierName: "CureMed", contact: "9000000006" },
  { supplierId: 107, supplierName: "MedSupply", contact: "9000000007" },
  { supplierId: 108, supplierName: "Global Pharma", contact: "9000000008" },
  { supplierId: 109, supplierName: "SafeMeds", contact: "9000000009" },
  { supplierId: 110, supplierName: "Prime Healthcare", contact: "9000000010" }
])
```

---

## 3. Medicines — 10 Documents

```javascript
db.medicines.insertMany([
  {
    medicineId: 1,
    medicineName: "Paracetamol",
    supplierId: 101,
    price: 20.0,
    stock: 100,
    expiryDate: ISODate("2027-06-30")
  },
  {
    medicineId: 2,
    medicineName: "Amoxicillin",
    supplierId: 102,
    price: 50.0,
    stock: 60,
    expiryDate: ISODate("2027-08-31")
  },
  {
    medicineId: 3,
    medicineName: "Cetirizine",
    supplierId: 103,
    price: 15.0,
    stock: 80,
    expiryDate: ISODate("2028-01-31")
  },
  {
    medicineId: 4,
    medicineName: "Ibuprofen",
    supplierId: 104,
    price: 30.0,
    stock: 45,
    expiryDate: ISODate("2027-09-30")
  },
  {
    medicineId: 5,
    medicineName: "Azithromycin",
    supplierId: 105,
    price: 75.0,
    stock: 90,
    expiryDate: ISODate("2028-03-31")
  },
  {
    medicineId: 6,
    medicineName: "Omeprazole",
    supplierId: 106,
    price: 40.0,
    stock: 55,
    expiryDate: ISODate("2027-11-30")
  },
  {
    medicineId: 7,
    medicineName: "Metformin",
    supplierId: 107,
    price: 35.0,
    stock: 120,
    expiryDate: ISODate("2028-02-29")
  },
  {
    medicineId: 8,
    medicineName: "Aspirin",
    supplierId: 108,
    price: 25.0,
    stock: 65,
    expiryDate: ISODate("2027-12-31")
  },
  {
    medicineId: 9,
    medicineName: "Pantoprazole",
    supplierId: 109,
    price: 45.0,
    stock: 40,
    expiryDate: ISODate("2028-04-30")
  },
  {
    medicineId: 10,
    medicineName: "Loratadine",
    supplierId: 110,
    price: 28.0,
    stock: 75,
    expiryDate: ISODate("2028-05-31")
  }
])
```

---

## 4. Pharmacists — 10 Documents

```javascript
db.pharmacists.insertMany([
  { pharmacistId: 1, name: "Dr. Kumar", licenseNo: "LIC001" },
  { pharmacistId: 2, name: "Dr. Meera", licenseNo: "LIC002" },
  { pharmacistId: 3, name: "Dr. Ramesh", licenseNo: "LIC003" },
  { pharmacistId: 4, name: "Dr. Anita", licenseNo: "LIC004" },
  { pharmacistId: 5, name: "Dr. Vijay", licenseNo: "LIC005" },
  { pharmacistId: 6, name: "Dr. Sunita", licenseNo: "LIC006" },
  { pharmacistId: 7, name: "Dr. Karthik", licenseNo: "LIC007" },
  { pharmacistId: 8, name: "Dr. Rekha", licenseNo: "LIC008" },
  { pharmacistId: 9, name: "Dr. Arvind", licenseNo: "LIC009" },
  { pharmacistId: 10, name: "Dr. Nisha", licenseNo: "LIC010" }
])
```

---

## 5. Orders — 10 Documents

The `totalAmount` values are calculated as:

```text
quantity × medicine price
```

```javascript
db.orders.insertMany([
  {
    orderId: 1,
    customerId: 1,
    medicineId: 1,
    pharmacistId: 1,
    quantity: 3,
    totalAmount: 60.0,
    orderDate: ISODate("2026-01-10")
  },
  {
    orderId: 2,
    customerId: 2,
    medicineId: 2,
    pharmacistId: 2,
    quantity: 2,
    totalAmount: 100.0,
    orderDate: ISODate("2026-01-11")
  },
  {
    orderId: 3,
    customerId: 3,
    medicineId: 3,
    pharmacistId: 3,
    quantity: 5,
    totalAmount: 75.0,
    orderDate: ISODate("2026-01-12")
  },
  {
    orderId: 4,
    customerId: 4,
    medicineId: 4,
    pharmacistId: 4,
    quantity: 4,
    totalAmount: 120.0,
    orderDate: ISODate("2026-01-13")
  },
  {
    orderId: 5,
    customerId: 5,
    medicineId: 5,
    pharmacistId: 5,
    quantity: 2,
    totalAmount: 150.0,
    orderDate: ISODate("2026-01-14")
  },
  {
    orderId: 6,
    customerId: 6,
    medicineId: 6,
    pharmacistId: 6,
    quantity: 3,
    totalAmount: 120.0,
    orderDate: ISODate("2026-01-15")
  },
  {
    orderId: 7,
    customerId: 7,
    medicineId: 7,
    pharmacistId: 7,
    quantity: 4,
    totalAmount: 140.0,
    orderDate: ISODate("2026-01-16")
  },
  {
    orderId: 8,
    customerId: 8,
    medicineId: 8,
    pharmacistId: 8,
    quantity: 3,
    totalAmount: 75.0,
    orderDate: ISODate("2026-01-17")
  },
  {
    orderId: 9,
    customerId: 9,
    medicineId: 9,
    pharmacistId: 9,
    quantity: 2,
    totalAmount: 90.0,
    orderDate: ISODate("2026-01-18")
  },
  {
    orderId: 10,
    customerId: 10,
    medicineId: 10,
    pharmacistId: 10,
    quantity: 5,
    totalAmount: 140.0,
    orderDate: ISODate("2026-01-19")
  }
])
```

---

# d. Aggregation Queries

## i. List All Orders with Customer Name and Medicine Name

We start from `orders` and perform two joins:

```text
Orders → Customers
Orders → Medicines
```

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "customerId",
      as: "customer"
    }
  },
  {
    $unwind: "$customer"
  },
  {
    $lookup: {
      from: "medicines",
      localField: "medicineId",
      foreignField: "medicineId",
      as: "medicine"
    }
  },
  {
    $unwind: "$medicine"
  },
  {
    $project: {
      _id: 0,
      orderId: 1,
      customerName: "$customer.name",
      medicineName: "$medicine.medicineName",
      quantity: 1,
      totalAmount: 1,
      orderDate: 1
    }
  }
])
```

### Important pattern

```text
$lookup → $unwind → $lookup → $unwind → $project
```

---

# ii. Total Sales per Medicine

We need to:

1. Join orders with medicines
2. Group by medicine
3. Add all order amounts

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "medicines",
      localField: "medicineId",
      foreignField: "medicineId",
      as: "medicine"
    }
  },
  {
    $unwind: "$medicine"
  },
  {
    $group: {
      _id: "$medicine.medicineName",
      totalSales: { $sum: "$totalAmount" }
    }
  },
  {
    $project: {
      _id: 0,
      medicine: "$_id",
      totalSales: 1
    }
  },
  {
    $sort: {
      totalSales: -1
    }
  }
])
```

### Important operators

```text
$lookup → JOIN
$group → GROUP BY
$sum → TOTAL
$sort → ORDER BY
```

---

# iii. Supplier-wise Revenue

Relationship:

```text
Orders
   ↓ medicineId
Medicines
   ↓ supplierId
Suppliers
```

Therefore, two `$lookup` operations are required.

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "medicines",
      localField: "medicineId",
      foreignField: "medicineId",
      as: "medicine"
    }
  },
  {
    $unwind: "$medicine"
  },
  {
    $lookup: {
      from: "suppliers",
      localField: "medicine.supplierId",
      foreignField: "supplierId",
      as: "supplier"
    }
  },
  {
    $unwind: "$supplier"
  },
  {
    $group: {
      _id: "$supplier.supplierName",
      revenue: { $sum: "$totalAmount" }
    }
  },
  {
    $project: {
      _id: 0,
      supplier: "$_id",
      revenue: 1
    }
  },
  {
    $sort: {
      revenue: -1
    }
  }
])
```

---

# iv. Medicines Below Stock Threshold

Display medicines having **less than 70 units** in stock.

```javascript
db.medicines.aggregate([
  {
    $match: {
      stock: { $lt: 70 }
    }
  },
  {
    $project: {
      _id: 0,
      medicineId: 1,
      medicineName: 1,
      stock: 1
    }
  },
  {
    $sort: {
      stock: 1
    }
  }
])
```

### Important

```javascript
$match
```

is equivalent to a SQL:

```sql
WHERE
```

So:

```javascript
{$match:{stock:{$lt:70}}}
```

means:

```sql
WHERE stock < 70
```

---

# v. Customer Order Summary

Display each customer's:

* Name
* Number of orders
* Total amount spent

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "customerId",
      as: "customer"
    }
  },
  {
    $unwind: "$customer"
  },
  {
    $group: {
      _id: "$customer.customerId",
      customerName: { $first: "$customer.name" },
      totalOrders: { $sum: 1 },
      totalSpent: { $sum: "$totalAmount" }
    }
  },
  {
    $project: {
      _id: 0,
      customerName: 1,
      totalOrders: 1,
      totalSpent: 1
    }
  },
  {
    $sort: {
      totalSpent: -1
    }
  }
])
```

---

# Quick Revision

For this experiment, remember the following MongoDB operators:

| Operator   | Meaning         |
| ---------- | --------------- |
| `$lookup`  | JOIN            |
| `$unwind`  | Expand array    |
| `$match`   | WHERE / Filter  |
| `$group`   | GROUP BY        |
| `$sum`     | Calculate total |
| `$project` | SELECT fields   |
| `$sort`    | ORDER BY        |

### Aggregation patterns

**1. Orders + customer + medicine**

```text
$lookup → $unwind → $lookup → $unwind → $project
```

**2. Total sales**

```text
$lookup → $unwind → $group → $project → $sort
```

**3. Supplier revenue**

```text
$lookup → $unwind → $lookup → $unwind → $group → $project → $sort
```

**4. Low stock**

```text
$match → $project → $sort
```

**5. Customer summary**

```text
$lookup → $unwind → $group → $project → $sort
```

## Result

The pharmacy database was successfully created with schema validation for customers, medicines, suppliers, orders, and pharmacists. Ten documents were inserted into each collection, and multi-collection aggregation queries were performed to generate order, medicine sales, supplier revenue, stock, and customer summary reports.
