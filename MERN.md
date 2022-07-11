## DEPLOYING AN APP ON MERN STACK

## Step one: Install Node

  - update ubuntu: `sudo app upgrade` 
  - upgrade ubuntu: `sudo app update` 
  - get location of node software: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -` 
  - install node.js and npm: `sudo apt-get install -y nodejs` 
  - verify node version: `node -v`
  - verify npm version: `npm -v` 
 
## Step two: Set Up The Project

  - create an empty directory called `Todo`: `mkdir Todo` 
  - create a `React app - client ` inside `Todo`: `npx create-react-app client` 
  
  ![create_react_client](https://user-images.githubusercontent.com/92983658/178269789-870ba7b1-d769-46a3-a082-727296084d38.png)
  
  - creaate a folder for backend called `server` : `mkdir server` 
  - initialise `package.json` inside `server` : 
      - `cd server` 
      - `npm init` 
    
    ![initialise_json_in_server](https://user-images.githubusercontent.com/92983658/178271884-a8d5a043-b0b1-45d2-a01b-db44cf60c275.png)
    
  - install dependencies: `npm install mongodb express cors dotenv` 
    - installs: express, mongoDB, dotenv and cors
  
  ![install_dependencies_in_server](https://user-images.githubusercontent.com/92983658/178272447-7c4edcff-7ea5-4bfc-a9cd-9ef9cb3d3f12.png)

  - create `server.js` in `server` folder : `touch server.js` 
  - paste the following code inside the `server.js` file:
    
      - `nano server.js` 
      - paste the following:
   
 ```

const express = require("express");
const app = express();
const cors = require("cors");
require("dotenv").config({ path: "./config.env" });
const port = process.env.PORT || 5000;
app.use(cors());
app.use(express.json());
app.use(require("./routes/record"));
// get driver connection
const dbo = require("./db/conn");
 
app.listen(port, () => {
  // perform a database connection when server starts
  dbo.connectToServer(function (err) {
    if (err) console.error(err);
 
  });
  console.log(`Server is running on port: ${port}`);
});

```
   
 
  ![server_js_file](https://user-images.githubusercontent.com/92983658/178275818-234b5b81-6923-4360-b076-aa7a200cdaf8.png)

## Step Three: Connect To Database

- locate Database connection string
- inside server folder, create a `config.env` file: `cd server` then `touch config.env`
- inside `config.env` file, assign the connection string to a new DB variable.:
  - nano `config.env`
  - paste the following code inside. Replace <password> with database password:
 
```
 ATLAS_URI=mongodb+srv://<username>:<password>@sandbox.jadwj.mongodb.net/employees?retryWrites=true&w=majority
 PORT=5000
  
```
  
  ![paste_dbstring_port](https://user-images.githubusercontent.com/92983658/178277257-a67d858d-45f1-424d-a098-e5daf37a3f6f.png)

- inside the `server` folder, create a new folder `db` : `mkdir db`
- indside `db` create a new file `conn.js` : `touch conn.js`
- inside `conn.js` paste the following:
  
 ```
  const { MongoClient } = require("mongodb");
const Db = process.env.ATLAS_URI;
const client = new MongoClient(Db, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
 
var _db;
 
module.exports = {
  connectToServer: function (callback) {
    client.connect(function (err, db) {
      // Verify we got a good "db" object
      if (db)
      {
        _db = db.db("employees");
        console.log("Successfully connected to MongoDB."); 
      }
      return callback(err);
         });
  },
 
  getDb: function () {
    return _db;
  },
};
  
 ```
  
## Step Four: Server API Endpoints
  - inside `server` create a directory `routes`. create a file `record.js` inside the `routes` directory: 
    - `cd server`
    - ` mkdir routes`
    - `cd routes`
    - `touch record.js`
  - add the following lines of code to `record.js`:
  ```
  
  const express = require("express");
 
// recordRoutes is an instance of the express router.
// We use it to define our routes.
// The router will be added as a middleware and will take control of requests starting with path /record.
const recordRoutes = express.Router();
 
// This will help us connect to the database
const dbo = require("../db/conn");
 
// This help convert the id from string to ObjectId for the _id.
const ObjectId = require("mongodb").ObjectId;
 
 
// This section will help you get a list of all the records.
recordRoutes.route("/record").get(function (req, res) {
 let db_connect = dbo.getDb("employees");
 db_connect
   .collection("records")
   .find({})
   .toArray(function (err, result) {
     if (err) throw err;
     res.json(result);
   });
});
 
// This section will help you get a single record by id
recordRoutes.route("/record/:id").get(function (req, res) {
 let db_connect = dbo.getDb();
 let myquery = { _id: ObjectId( req.params.id )};
 db_connect
     .collection("records")
     .findOne(myquery, function (err, result) {
       if (err) throw err;
       res.json(result);
     });
});
 
// This section will help you create a new record.
recordRoutes.route("/record/add").post(function (req, response) {
 let db_connect = dbo.getDb();
 let myobj = {
   name: req.body.name,
   position: req.body.position,
   level: req.body.level,
 };
 db_connect.collection("records").insertOne(myobj, function (err, res) {
   if (err) throw err;
   response.json(res);
 });
});
 
// This section will help you update a record by id.
recordRoutes.route("/update/:id").post(function (req, response) {
 let db_connect = dbo.getDb(); 
 let myquery = { _id: ObjectId( req.params.id )}; 
 let newvalues = {   
   $set: {     
     name: req.body.name,    
     position: req.body.position,     
     level: req.body.level,   
   }, 
  }
});
 
// This section will help you delete a record
recordRoutes.route("/:id").delete((req, response) => {
 let db_connect = dbo.getDb();
 let myquery = { _id: ObjectId( req.params.id )};
 db_connect.collection("records").deleteOne(myquery, function (err, obj) {
   if (err) throw err;
   console.log("1 document deleted");
   response.json(obj);
 });
});
 
module.exports = recordRoutes;
  
```
  
  ![api_end_points](https://user-images.githubusercontent.com/92983658/178283729-1c0cf00b-b5f5-4802-91a4-86619c30aac2.png)

  - in EC2 security group for server, create a new inbound security rule to open TCP port 5000
  - confirm server and database connection:
    - `node server.js`
  
  ![confirm_server_DB_connection](https://user-images.githubusercontent.com/92983658/178285333-363004fd-a0c4-482a-907f-c49c7302769c.png)

  
  
## Step Five: Setting Up React App
## Step Six: Create Components
## Step Seven: Connect Front End To Back End.
