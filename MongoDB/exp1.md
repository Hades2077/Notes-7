
# NoSQL Lab — Experiment 1

## Employee CRUD Operations using MongoDB

### Aim

Create an Employee collection containing Employee ID, Name, Age, Gender, Department, Salary and Address, and perform CRUD operations using MongoDB.

---

## a. Insert Employee Details

```javascript
use employeeDB

db.employees.insertMany([
  {
    empId: "E001",
    name: "Anjali",
    age: 35,
    gender: "Female",
    department: "HR",
    salary: 48000,
    address: "Bangalore"
  },
  {
    empId: "E002",
    name: "Rakesh",
    age: 42,
    gender: "Male",
    department: "IT",
    salary: 62000,
    address: "Mysore"
  },
  {
    empId: "E003",
    name: "Divya",
    age: 29,
    gender: "Female",
    department: "Marketing",
    salary: 45000,
    address: "Chennai"
  }
])
```

---

## b. Retrieve Employees Whose Age is Less Than or Equal to 40

```javascript
db.employees.find({ age: { $lte: 40 } })
```

**`$lte` = Less Than or Equal To**

---

## c. Display Employees Whose Salary is Less Than 50,000

```javascript
db.employees.find({ salary: { $lt: 50000 } })
```

**`$lt` = Less Than**

---

## d. Demonstrate Update Operation

Update the salary of employee `E003` to `55,000`.

```javascript
db.employees.updateOne(
  { empId: "E003" },
  { $set: { salary: 55000 } }
)
```

**`updateOne()`** updates the first matching document.

**`$set`** changes the specified field.

---

## e. Remove Employees Belonging to a Particular Department

Remove all employees belonging to the HR department.

```javascript
db.employees.deleteMany({
  department: "HR"
})
```

**`deleteMany()`** removes all documents matching the condition.

---

]
