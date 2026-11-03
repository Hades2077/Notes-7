# Experiment 6: MongoDB Sharding Performance Test (Windows Guide)

This guide takes you through the setup from scratch, preventing port conflicts, folder permission errors, and `mongos` routing issues on Windows.

---

## Pre-Flight Check: Clean Slate & Kill Port Conflicts

Before starting, the default Windows MongoDB service must be stopped because it occupies port `27017`.

Open **PowerShell as Administrator** and execute:

```powershell
# 1. Stop default Windows background service if running
net stop MongoDB

# 2. Terminate any lingering mongod/mongos instances
taskkill /F /IM mongod.exe /T 2>$null
taskkill /F /IM mongos.exe /T 2>$null

# 3. Clean and recreate cluster data directories on drive C
Remove-Item -Recurse -Force "C:\data\configdb", "C:\data\shard1", "C:\data\shard2" -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Path "C:\data\configdb", "C:\data\shard1", "C:\data\shard2" -Force
```

---

## Cluster Architecture Overview

| Component | Port | Process | Data Directory | Replica Set Name |
| :--- | :--- | :--- | :--- | :--- |
| **Config Server** | `27019` | `mongod` | `C:\data\configdb` | `configReplSet` |
| **Shard 1 Server** | `27018` | `mongod` | `C:\data\shard1` | `shard1Repl` |
| **Shard 2 Server** | `27020` | `mongod` | `C:\data\shard2` | `shard2Repl` |
| **Query Router** | `27017` | `mongos` | *(In-Memory Router)* | N/A |

---

## Step-by-Step Terminal Execution

You will use **6 separate PowerShell windows**. 
Keep Terminals 1, 2, 3, and 5 open and running throughout the test.

---

### Terminal 1: Launch Config Server Daemon
Open PowerShell and run:
```powershell
mongod --configsvr --replSet configReplSet --dbpath "C:\data\configdb" --port 27019
```
*Leave this terminal open. It will continuously display server logs.*

---

### Terminal 2: Launch Shard 1 Daemon
Open a new PowerShell window and run:
```powershell
mongod --shardsvr --replSet shard1Repl --dbpath "C:\data\shard1" --port 27018
```
*Leave this terminal open.*

---

### Terminal 3: Launch Shard 2 Daemon
Open a new PowerShell window and run:
```powershell
mongod --shardsvr --replSet shard2Repl --dbpath "C:\data\shard2" --port 27020
```
*Leave this terminal open.*

---

### Terminal 4: Initialize Replica Sets (Configuration Terminal)
Open a new PowerShell window. You will connect to each port one by one and initialize its replica set.

#### 4A. Initialize Config Server
```powershell
mongosh --port 27019
```
Inside the `test>` shell prompt, execute:
```javascript
rs.initiate({
  _id: "configReplSet",
  configsvr: true,
  members: [ { _id: 0, host: "localhost:27019" } ]
})
```
Wait for `{ ok: 1 }`, then exit:
```javascript
exit
```

#### 4B. Initialize Shard 1
Back in the PowerShell prompt:
```powershell
mongosh --port 27018
```
Inside the `test>` shell prompt, execute:
```javascript
rs.initiate({
  _id: "shard1Repl",
  members: [ { _id: 0, host: "localhost:27018" } ]
})
```
Wait for `{ ok: 1 }`, then exit:
```javascript
exit
```

#### 4C. Initialize Shard 2
Back in the PowerShell prompt:
```powershell
mongosh --port 27020
```
Inside the `test>` shell prompt, execute:
```javascript
rs.initiate({
  _id: "shard2Repl",
  members: [ { _id: 0, host: "localhost:27020" } ]
})
```
Wait for `{ ok: 1 }`, then exit:
```javascript
exit
```
*(You may close Terminal 4 now).*

---

### Terminal 5: Launch mongos Router Daemon
Now that the Config Server and Shard nodes are initialized, launch the router.

Open a new PowerShell window and run:
```powershell
mongos --configdb configReplSet/localhost:27019 --port 27017
```
*Leave this terminal open.*

---

### Terminal 6: Client Workstation (Running Queries & Performance Tests)

Open a new PowerShell window. Connect to the `mongos` router:
```powershell
mongosh --port 27017
```

> **Verification:** Your prompt should now display `mongos>` or `[direct: mongos] test>`. If it shows `test>` without mentioning `mongos`, verify that Terminal 5 is still running.

Execute the following database commands in sequence:

#### Step 1: Register Both Shards with Router
```javascript
sh.addShard("shard1Repl/localhost:27018")
sh.addShard("shard2Repl/localhost:27020")
```
Confirm with:
```javascript
sh.status()
```
*(You will see both `shard1Repl` and `shard2Repl` listed under the `shards` section).*

---

#### Step 2: Switch to Database and Populate Unsharded Collection
Use batch insertion with `insertMany` in chunks of 5,000 to insert data efficiently:

```javascript
use perfTestDB

db.movies_unsharded.drop()

print("Inserting 50,000 documents into unsharded collection...")
for (let b = 0; b < 10; b++) {
  let docs = [];
  for (let i = 0; i < 5000; i++) {
    let id = b * 5000 + i;
    docs.push({
      title: "Movie " + id,
      genre: (id % 5 === 0) ? "Action" : "Drama",
      year: 2000 + (id % 20),
      rating: Math.random() * 10
    });
  }
  db.movies_unsharded.insertMany(docs);
}
```

---

#### Step 3: Enable Sharding and Populate Sharded Collection
Enable sharding on the database, shard the collection on `{ year: 1 }`, and populate it:

```javascript
sh.enableSharding("perfTestDB")

db.movies_sharded.drop()

sh.shardCollection("perfTestDB.movies_sharded", { year: 1 })

print("Inserting 50,000 documents into sharded collection...")
for (let b = 0; b < 10; b++) {
  let docs = [];
  for (let i = 0; i < 5000; i++) {
    let id = b * 5000 + i;
    docs.push({
      title: "Movie " + id,
      genre: (id % 5 === 0) ? "Action" : "Drama",
      year: 2000 + (id % 20),
      rating: Math.random() * 10
    });
  }
  db.movies_sharded.insertMany(docs);
}
```

---

#### Step 4: Compare Performance using `explain("executionStats")`

##### Test A: Equality Query (`year = 2010`)
```javascript
// Unsharded Query Execution
db.movies_unsharded.find({ year: 2010 }).explain("executionStats").executionStats

// Sharded Query Execution (Targeted Shard Routing)
db.movies_sharded.find({ year: 2010 }).explain("executionStats").executionStats
```

##### Test B: Range Query (`2010 <= year < 2015`)
```javascript
// Unsharded Query Execution
db.movies_unsharded.find({ year: { $gte: 2010, $lt: 2015 } }).explain("executionStats").executionStats

// Sharded Query Execution
db.movies_sharded.find({ year: { $gte: 2010, $lt: 2015 } }).explain("executionStats").executionStats
```

##### Test C: Aggregation Pipeline
```javascript
// Unsharded Aggregation
db.movies_unsharded.aggregate([
  { $match: { year: { $gte: 2010, $lt: 2015 } } },
  { $group: { _id: "$genre", avgRating: { $avg: "$rating" } } }
]).explain("executionStats")

// Sharded Aggregation (Distributed execution across shards)
db.movies_sharded.aggregate([
  { $match: { year: { $gte: 2010, $lt: 2015 } } },
  { $group: { _id: "$genre", avgRating: { $avg: "$rating" } } }
]).explain("executionStats")
```

---

## How to Explain the Output to the Examiner

Show the examiner the differences in the `executionStats` output block:

| Metric | Unsharded Collection | Sharded Collection | Explanation |
| :--- | :--- | :--- | :--- |
| **Stage** | `COLLSCAN` | `SHARD_MERGE` $\rightarrow$ `IXSCAN` / Targeted Shard Scan | The unsharded query scans the entire collection sequentially on one disk. The sharded query is routed directly to the specific shard holding chunks for that year range. |
| **`totalDocsExamined`** | High (scans all $50,000$ documents) | Significantly lower (only inspects documents on targeted shard chunks) | Prunes unneeded partitions before reading disk blocks. |
| **`executionTimeMillis`** | Higher execution latency | Lower execution latency | Parallel processing and reduced data scan footprint. |
