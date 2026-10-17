# Lesson 003 — Installation: Local MongoDB, Compass and mongosh

[← Section 1](./README.md) · [Full curriculum](../README.md)

## What you will learn
MongoDB Server stores data; **mongosh** runs commands; **Compass** is a graphical client. You can use either client to inspect the same database.

## 1. How the tools connect
```text
mongosh (terminal) ──┐
                     ├── MongoDB Server (localhost:27017) ── databases
Compass (GUI) ───────┘
```
Installing Compass alone does **not** install or start MongoDB Server.

## 2. Install locally
Use the [MongoDB Community Server installation guide](https://www.mongodb.com/docs/manual/installation/) for your exact OS and version. On Ubuntu, follow the [official Ubuntu instructions](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/) matching your Ubuntu release. The repository and package names depend on the release; do not blindly copy commands for a different version.

Install [mongosh](https://www.mongodb.com/docs/mongodb-shell/install/) and optionally [Compass](https://www.mongodb.com/docs/compass/current/install/) separately. On supported Linux installations using systemd, the service is commonly named `mongod`:

```bash
sudo systemctl start mongod
sudo systemctl status mongod
```
If the service is missing, check that the server package was installed correctly. On Windows/macOS, start the server using the method described in the corresponding official installation guide.

## 3. Verify with mongosh
```bash
mongosh "mongodb://127.0.0.1:27017"
```
Inside **mongosh**, run:
```js
db.runCommand({ ping: 1 })
use learningDB
db.students.insertOne({ name: "Asha", course: "MongoDB" })
db.students.find()
show collections
```
Expected: `ping` returns `ok: 1`; the insert acknowledges the write and returns an `insertedId`; the query displays your student.

**Why `127.0.0.1`?** It refers to your own computer. Port `27017` is the usual MongoDB default. Local deployments may use other ports.

## 4. Inspect the same data in Compass
Open Compass → New Connection → enter `mongodb://127.0.0.1:27017` → Connect → select `learningDB` → `students`. You should see the inserted document. Compass is a GUI, not another database.

## 5. Common issues
- **ECONNREFUSED:** Server is not running, port is wrong, or server is not listening on that address.
- **Authentication failed:** Credentials/authentication database may be wrong.
- **No database shown:** An empty database selected by `use` may not appear until you write data.
- **Command not found:** Install mongosh separately or verify your PATH.
- **Remote access:** Do not expose an unauthenticated MongoDB server to the public internet.

## Practice
Insert two students using mongosh, then find them in Compass. Stop the local server and predict what happens when you query again.

## Interview check
**Server vs Compass vs mongosh?** Server stores and processes data; Compass and mongosh are clients.

**References:** [Server installation](https://www.mongodb.com/docs/manual/installation/) · [mongosh](https://www.mongodb.com/docs/mongodb-shell/) · [Compass](https://www.mongodb.com/docs/compass/current/)
