# NoSQL Lab — Experiment 2

## 2. University Database using MongoDB

### Aim

Create a university database containing collections for **students, courses, departments, enrollments, and faculty** with schema validation. Insert sample data and perform aggregation queries using `$lookup`, `$match`, `$group`, `$project`, `$sort`, `$unwind`, and `$facet`.

---

## a. Switch to Database

```javascript
use universityDB
```

---

## b. Create Collections with Schema Validation

### 1. Students

```javascript
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["studentId", "name", "age", "deptId"],
      properties: {
        studentId: { bsonType: "int" },
        name: { bsonType: "string" },
        age: { bsonType: "int", minimum: 18 },
        deptId: { bsonType: "int" }
      }
    }
  }
})
```

### 2. Courses

```javascript
db.createCollection("courses", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["courseId", "courseName", "deptId"],
      properties: {
        courseId: { bsonType: "int" },
        courseName: { bsonType: "string" },
        deptId: { bsonType: "int" }
      }
    }
  }
})
```

### 3. Departments

```javascript
db.createCollection("departments", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["deptId", "deptName"],
      properties: {
        deptId: { bsonType: "int" },
        deptName: { bsonType: "string" }
      }
    }
  }
})
```

### 4. Faculty

```javascript
db.createCollection("faculty", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["facultyId", "name", "deptId", "joiningDate"],
      properties: {
        facultyId: { bsonType: "int" },
        name: { bsonType: "string" },
        deptId: { bsonType: "int" },
        joiningDate: { bsonType: "date" }
      }
    }
  }
})
```

### 5. Enrollments

```javascript
db.createCollection("enrollments", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["studentId", "courseId", "enrollDate"],
      properties: {
        studentId: { bsonType: "int" },
        courseId: { bsonType: "int" },
        enrollDate: { bsonType: "date" }
      }
    }
  }
})
```

---

# c. Insert Sample Data

## Students

```javascript
db.students.insertMany([
  { studentId: 1, name: "Arjun", age: 20, deptId: 101 },
  { studentId: 2, name: "Priya", age: 21, deptId: 102 },
  { studentId: 3, name: "Ravi", age: 22, deptId: 101 },
  { studentId: 4, name: "Meena", age: 20, deptId: 103 },
  { studentId: 5, name: "Amit", age: 23, deptId: 101 },
  { studentId: 6, name: "Kiran", age: 19, deptId: 102 },
  { studentId: 7, name: "Sita", age: 20, deptId: 103 },
  { studentId: 8, name: "Raj", age: 21, deptId: 104 },
  { studentId: 9, name: "Anita", age: 22, deptId: 101 },
  { studentId: 10, name: "Vikram", age: 23, deptId: 104 }
])
```

## Departments

```javascript
db.departments.insertMany([
  { deptId: 101, deptName: "Computer Science" },
  { deptId: 102, deptName: "Electronics" },
  { deptId: 103, deptName: "Mechanical" },
  { deptId: 104, deptName: "Civil" },
  { deptId: 105, deptName: "Architecture" },
  { deptId: 106, deptName: "Biotechnology" },
  { deptId: 107, deptName: "Mathematics" },
  { deptId: 108, deptName: "Physics" },
  { deptId: 109, deptName: "Chemistry" },
  { deptId: 110, deptName: "English" }
])
```

## Courses

```javascript
db.courses.insertMany([
  { courseId: 1, courseName: "DBMS", deptId: 101 },
  { courseId: 2, courseName: "Data Structures", deptId: 101 },
  { courseId: 3, courseName: "Circuits", deptId: 102 },
  { courseId: 4, courseName: "Thermodynamics", deptId: 103 },
  { courseId: 5, courseName: "Surveying", deptId: 104 },
  { courseId: 6, courseName: "Microbiology", deptId: 106 },
  { courseId: 7, courseName: "Calculus", deptId: 107 },
  { courseId: 8, courseName: "Optics", deptId: 108 },
  { courseId: 9, courseName: "Organic Chemistry", deptId: 109 },
  { courseId: 10, courseName: "Literature", deptId: 110 }
])
```

## Faculty

```javascript
db.faculty.insertMany([
  { facultyId: 1, name: "Dr. Kumar", deptId: 101, joiningDate: ISODate("2018-06-10") },
  { facultyId: 2, name: "Dr. Meera", deptId: 102, joiningDate: ISODate("2017-07-15") },
  { facultyId: 3, name: "Dr. Ramesh", deptId: 103, joiningDate: ISODate("2019-03-20") },
  { facultyId: 4, name: "Dr. Anita", deptId: 104, joiningDate: ISODate("2016-08-25") },
  { facultyId: 5, name: "Dr. Vijay", deptId: 105, joiningDate: ISODate("2020-01-12") },
  { facultyId: 6, name: "Dr. Sunita", deptId: 106, joiningDate: ISODate("2021-09-05") },
  { facultyId: 7, name: "Dr. Karthik", deptId: 107, joiningDate: ISODate("2015-11-11") },
  { facultyId: 8, name: "Dr. Rekha", deptId: 108, joiningDate: ISODate("2019-02-14") },
  { facultyId: 9, name: "Dr. Arvind", deptId: 109, joiningDate: ISODate("2020-05-18") },
  { facultyId: 10, name: "Dr. Nisha", deptId: 110, joiningDate: ISODate("2022-04-21") }
])
```

## Enrollments

```javascript
db.enrollments.insertMany([
  { studentId: 1, courseId: 1, enrollDate: ISODate("2023-08-01") },
  { studentId: 1, courseId: 2, enrollDate: ISODate("2023-08-02") },
  { studentId: 2, courseId: 3, enrollDate: ISODate("2023-08-03") },
  { studentId: 3, courseId: 1, enrollDate: ISODate("2023-08-04") },
  { studentId: 4, courseId: 4, enrollDate: ISODate("2023-08-05") },
  { studentId: 5, courseId: 2, enrollDate: ISODate("2023-08-06") },
  { studentId: 6, courseId: 3, enrollDate: ISODate("2023-08-07") },
  { studentId: 7, courseId: 4, enrollDate: ISODate("2023-08-08") },
  { studentId: 8, courseId: 5, enrollDate: ISODate("2023-08-09") },
  { studentId: 9, courseId: 1, enrollDate: ISODate("2023-08-10") }
])
```

---

# d. Aggregation Queries

## i. List Each Student with Department Name

```javascript
db.students.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "deptId",
      foreignField: "deptId",
      as: "deptInfo"
    }
  },
  {
    $unwind: "$deptInfo"
  },
  {
    $project: {
      _id: 0,
      studentId: 1,
      name: 1,
      department: "$deptInfo.deptName"
    }
  }
])
```

---

## ii. Show Each Student's Enrolled Courses with Course Names

```javascript
db.students.aggregate([
  {
    $lookup: {
      from: "enrollments",
      localField: "studentId",
      foreignField: "studentId",
      as: "enrollments"
    }
  },
  {
    $unwind: "$enrollments"
  },
  {
    $lookup: {
      from: "courses",
      localField: "enrollments.courseId",
      foreignField: "courseId",
      as: "courseInfo"
    }
  },
  {
    $unwind: "$courseInfo"
  },
  {
    $project: {
      _id: 0,
      name: 1,
      course: "$courseInfo.courseName",
      enrollDate: "$enrollments.enrollDate"
    }
  }
])
```

---

## iii. Number of Students per Department

```javascript
db.students.aggregate([
  {
    $group: {
      _id: "$deptId",
      totalStudents: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "departments",
      localField: "_id",
      foreignField: "deptId",
      as: "deptInfo"
    }
  },
  {
    $unwind: "$deptInfo"
  },
  {
    $project: {
      _id: 0,
      department: "$deptInfo.deptName",
      totalStudents: 1
    }
  },
  {
    $sort: {
      totalStudents: -1
    }
  }
])
```

---

## iv. Faculty List with Their Courses and Departments

```javascript
db.faculty.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "deptId",
      foreignField: "deptId",
      as: "deptInfo"
    }
  },
  {
    $unwind: "$deptInfo"
  },
  {
    $lookup: {
      from: "courses",
      localField: "deptId",
      foreignField: "deptId",
      as: "coursesTaught"
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      department: "$deptInfo.deptName",
      coursesTaught: "$coursesTaught.courseName"
    }
  }
])
```

---

## v. Department Summary

Display total students, faculty, and courses for each department.

```javascript
db.departments.aggregate([
  {
    $lookup: {
      from: "students",
      localField: "deptId",
      foreignField: "deptId",
      as: "studentsList"
    }
  },
  {
    $lookup: {
      from: "faculty",
      localField: "deptId",
      foreignField: "deptId",
      as: "facultyList"
    }
  },
  {
    $lookup: {
      from: "courses",
      localField: "deptId",
      foreignField: "deptId",
      as: "coursesList"
    }
  },
  {
    $project: {
      _id: 0,
      deptName: 1,
      totalStudents: { $size: "$studentsList" },
      totalFaculty: { $size: "$facultyList" },
      totalCourses: { $size: "$coursesList" }
    }
  }
])
```

---

# Additional Aggregation Operators

The question explicitly mentions `$match` and `$facet`. The supplied queries don't actually demonstrate them, so these can be used to demonstrate those operators.

## `$match` — Filter Students

```javascript
db.students.aggregate([
  {
    $match: {
      age: { $gt: 20 }
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      age: 1
    }
  }
])
```

## `$facet` — Generate Multiple Reports

```javascript
db.students.aggregate([
  {
    $facet: {
      olderStudents: [
        { $match: { age: { $gt: 20 } } },
        { $project: { _id: 0, name: 1, age: 1 } }
      ],

      departmentCount: [
        {
          $group: {
            _id: "$deptId",
            count: { $sum: 1 }
          }
        }
      ]
    }
  }
])
```

---

# Result / Conclusion

The university database was successfully created in MongoDB with schema validation. Sample documents were inserted into the five collections, and aggregation pipelines using `$lookup`, `$unwind`, `$match`, `$group`, `$project`, `$sort`, and `$facet` were performed to generate student, course, faculty, and department reports.
