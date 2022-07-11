## DEPLOYING AN APP ON MERN STACK

## Step one: Install Node

  - update ubuntu: `sudo apt upgrade` 
  - upgrade ubuntu: `sudo apt update` 
  - get location of node software: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -` 
  - install node.js and npm: `sudo apt-get install -y nodejs` 
  - verify node version: `node -v`
  - verify npm version: `npm -v` 
 
## Step two: Set Up The Project

  - create an empty directory called `Todo`: `mkdir Todo` 
  - create a `React app - client ` inside `Todo`: `npx create-react-app client` 
  
  ![create_react_client](https://user-images.githubusercontent.com/92983658/178269789-870ba7b1-d769-46a3-a082-727296084d38.png)
  
   
  - initialise `package.json` inside `Todo` : 
       
      - `npm init` 
    
  ![init_json](https://user-images.githubusercontent.com/92983658/178304102-8c1b2d68-64d7-4300-b995-46b25bebd5d3.png)

    
  - install dependencies - express, mongoDB, dotenv and cors: `npm install mongodb express cors dotenv` 
  - create `index.js` in `Todo` folder : `touch index.js` 
  - paste the following code inside the `index.js` file: 
      - `vim index.js` 
      - paste the following:
   
 ```

const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
   
 ![index_js](https://user-images.githubusercontent.com/92983658/178305814-62d970f3-5f6c-458f-aecd-e37ae99276cb.png)

- in EC2 security group for server, create a new inbound security rule to open TCP port 5000
  - confirm server connection:
  - `node index.js`
  
![Screenshot 2022-07-11 at 16 58 04](https://user-images.githubusercontent.com/92983658/178306649-f6f9d92d-16a9-4e13-9ff7-3cc2cec2bf64.png)

- open browser and access server IP:
  - `http://<PublicIP-or-PublicDNS>:5000`

![browser](https://user-images.githubusercontent.com/92983658/178307348-02de6ff6-f8ef-4d02-93eb-675485c66baf.png)


## Step Three: Create Models

- install `mongoose` in `ToDo` directory : `npm install mongoose`
- create a new directory `models` : `mkdir models`
- change directory to `models` and create a file named `todo.js` : 
  - `cd models`
  -  `touch todo.js`
  -  open `todo.js` and paste the following code:

~~~

const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;

~~~

![Screenshot 2022-07-11 at 17 35 21](https://user-images.githubusercontent.com/92983658/178313903-5ffc9254-6039-46e2-a847-336d8601dfae.png)

## Step Three: Connect To Database

- locate Database connection string
- inside `Todo` directory, create a `.env` file: `touch config.env`
- inside the `.env` file, assign the connection string to a new DB variable.:
  - vim `.env`
  - paste the following code inside. Replace <password> with database password:
 
```
 DB=mongodb+srv://<username>:<password>@sandbox.jadwj.mongodb.net/employees?retryWrites=true&w=majority
  
```
  
  ![paste_dbstring_port](https://user-images.githubusercontent.com/92983658/178277257-a67d858d-45f1-424d-a098-e5daf37a3f6f.png)

- update `index.js` file
  - `vim index.js`
  - inside `index.js` paste the following:
  
~~~
 const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
  
~~~
 
- confirm database is connected : `node index.js`
  
  ![Screenshot 2022-07-11 at 17 50 07](https://user-images.githubusercontent.com/92983658/178316576-126a3f93-a070-4f37-bd4e-ed81bf4b23c9.png)

  
## Step Four: Server API Endpoints

  - inside `Todo` create a directory `routes`. create a file `record.js` inside the `routes` directory: 
    - ` mkdir routes`
    - `cd routes`
    - `touch api.js`
  - add the following lines of code to `api.js`:

  ```
  
  const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
  
```
  
  ![api_js](https://user-images.githubusercontent.com/92983658/178309460-be6da90f-1cdf-427f-960a-12387373b715.png)

 
  
## Step Five: Setting Up React App
## Step Six: Create Components
## Step Seven: Connect Front End To Back End.
