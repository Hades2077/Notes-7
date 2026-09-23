use employeeDB

// a. Insert 

db.employees.insertMany([
  {empId:"E001", name:"Anjali", age:35, gender:"Female", department:"HR", salary:48000, address:"Bangalore"},
  {empId:"E002", name:"Rakesh", age:42, gender:"Male", department:"IT", salary:62000, address:"Mysore"},
  {empId:"E003", name:"Divya", age:29, gender:"Female", department:"Marketing", salary:45000, address:"Chennai"}
])

// b. Age <= 40

db.employees.find({age:{$lte:40}})

// c. Salary < 50000

db.employees.find({salary:{$lt:50000}})

// d. Update

db.employees.updateOne(
  {empId:"E003"},
  {$set:{salary:55000}}
)

// e. Delete department

db.employees.deleteMany({department:"HR"})
