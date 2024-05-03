I will recap everything that I did to initialize and run this project in windows 
In the terminal of the folder in the vs code enter the following:
mkdir student-marks-app
cd student-marks-app
npm init -y
npm install express mongodb 
Create a new file app.js
MONGO_URI=mongodb://0.0.0.0:27017
PORT=3000
mongod

create app.js in student-marks folder and enter the following:
const express = require("express");
const { MongoClient } = require("mongodb");

const app = express();
const port = 3000;
const mongoUri = "mongodb://0.0.0.0:27017";
const client = new MongoClient(mongoUri);

app.use(express.json());

// Connect to MongoDB
async function main() {
  try {
    await client.connect();
    console.log("Connected to MongoDB");
    const db = client.db("student");
    const collection = db.collection("studentmarks");

    // Clear the collection
    await collection.deleteMany({});

    // Insert array of documents
    await collection.insertMany([
      {
        Name: "Alice",
        Roll_No: 1,
        WAD_Marks: 22,
        CC_Marks: 25,
        DSBDA_Marks: 26,
        CNS_Marks: 30,
        AI_marks: 31,
      },
      {
        Name: "Bob",
        Roll_No: 2,
        WAD_Marks: 20,
        CC_Marks: 23,
        DSBDA_Marks: 15,
        CNS_Marks: 10,
        AI_marks: 20,
      },
      // Add more documents as needed
    ]);

    // Display total count of documents and list all documents
    app.get("/documents", async (req, res) => {
      const docs = await collection.find({}).toArray();
      const count = await collection.countDocuments();
      res.send({ count, docs });
    });

    // List the names of students who got more than 20 marks in DSBDA
    app.get("/high-dsbda-marks", async (req, res) => {
      const students = await collection
        .find({ DSBDA_Marks: { $gt: 20 } })
        .project({ Name: 1, _id: 0 })
        .toArray();
      res.send(students);
    });

    // Update the marks of specified students by 10
    app.patch("/update-marks", async (req, res) => {
      const result = await collection.updateOne(
        { Name: "Alice" }, // Update Alice's marks
        {
          $inc: {
            WAD_Marks: 10,
            CC_Marks: 10,
            DSBDA_Marks: 10,
            CNS_Marks: 10,
            AI_marks: 10,
          },
        }
      );
      res.send(result);
    });

    // List the names who got more than 25 marks in all subjects
    app.get("/top-students", async (req, res) => {
      const students = await collection
        .find({
          WAD_Marks: { $gt: 25 },
          CC_Marks: { $gt: 25 },
          DSBDA_Marks: { $gt: 25 },
          CNS_Marks: { $gt: 25 },
          AI_marks: { $gt: 25 },
        })
        .project({ Name: 1, _id: 0 })
        .toArray();
      res.send(students);
    });

    // Remove specified student document
    app.get("/remove-student", async (req, res) => {
      const result = await collection.deleteOne({ Name: "Bob" }); // Remove Bob's document
      res.send(result);
    });

    // Display the Students data in Browser in tabular format.
    app.get("/table", async (req, res) => {
      const students = await collection.find({}).toArray();
      let html = '<table border="1">';
      html +=
        "<tr><th>Name</th><th>Roll_No</th><th>WAD_Marks</th><th>CC_Marks</th><th>DSBDA_Marks</th><th>CNS_Marks</th><th>AI_marks</th></tr>";
      students.forEach((student) => {
        html += `<tr><td>${student.Name}</td><td>${student.Roll_No}</td><td>${student.WAD_Marks}</td><td>${student.CC_Marks}</td><td>${student.DSBDA_Marks}</td><td>${student.CNS_Marks}</td><td>${student.AI_marks}</td></tr>`;
      });
      html += "</table>";
      res.send(html);
    });

    app.listen(port, () => console.log(`App listening on port ${port}`));
  } catch (e) {
    console.error(e);
  }
}

main().catch(console.error);

in a split terminal enter
node app.js

