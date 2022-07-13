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

## Step Four: Connect To Database

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

  
## Step Five: Server API Endpoints

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
  

## Step Six Test API Endpoints
  
  - open server : `node index.js`
  - open client in POSTMAN,
  - create a `POST` request and navigate to `http://localhost:5000/api/todos`
  - set header key to `content-type` and value to `application/json`
  - confirm there are no errors.
  
  ![POST](https://user-images.githubusercontent.com/92983658/178525916-770ccfce-b497-4075-938c-e64317e0f4ae.png)

  
- create a `GET `request to `http://localhost:5000/api/todos`

 ![GET](https://user-images.githubusercontent.com/92983658/178527588-3546e0b2-e141-421e-a427-609b2f750207.png)

- create a `DELETE` request
  
  ![DELETE](https://user-images.githubusercontent.com/92983658/178528180-6ab93dc7-08e2-4466-a2eb-abcff57c3a4f.png)
  

  
## Step Seven: Setting Up React App
  
  - create a `React app - client ` inside `Todo`: `npx create-react-app client` 
  
  ![create_react_client](https://user-images.githubusercontent.com/92983658/178269789-870ba7b1-d769-46a3-a082-727296084d38.png)
  
  - install dependencies `concurrently` and `nodemon`: 
    - concurrently : `npm install concurrently --save-dev`
    - nodemon: `npm install nodemon --save-dev`
  
  - in `Todo` directory, update the `scripts` section in `package.json` with the following code:
  
  ~~~
  
  "scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},

~~~

  
  - change directory to client: `cd client`
  - open `package.json` : `vim package.json`
  - add in the key-value pair : `"proxy": "http://localhost:5000"`
  
  ![key-value-add](https://user-images.githubusercontent.com/92983658/178489768-255aff1c-1b63-4155-9b49-49515f8b255e.png)

  - go back to `Todo` directory: `cd ..`
  - confirm that server is running on `localhost:3000` : `npm run dev`
  
  ![run_dev](https://user-images.githubusercontent.com/92983658/178541530-4562d10b-9243-4640-8032-c14c90520140.png)

  - open TCP port 3000 in EC2 

  
## Step Eight: Create React Components

- go to `client` directory : `cd client`
- create a `components` directory inside the `src` directory and create component files
  - `cd src && mkdir components && touch Input.js ListTodo.js Todo.js`
  
- open `input.js` file: `vim input.js`
- paste the following code:

```

import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input

```
  
- change to `client` directory : `cd ..`
- install `axios` : `npm install axios`
- go to `components` directory: `cd src/components`
- open `ListTodo.js` : `vim ListTodo.js`
- paste the following code:

  
```
  
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
  
```
  
- in `Todo.js` write the following:

```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;

```
  
- in`src` directory, open `App.js` : `vi App.js`
- paste the following:
  
``` 
  import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;

```
  

![Screenshot 2022-07-12 at 14 35 57](https://user-images.githubusercontent.com/92983658/178503321-36a93f03-ccb6-4fe8-912e-69173f42be22.png)

- in src directory, open `App.css` : `vim App.css`
- paste the following and exit

```
  
  .App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
  
 ```

 - in `src` directory, open `index.css` : `vi index.css`
 - paste the following code and exit:
 
 ```
  
  body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
  
```
  
- go to `Todo` directory and access the app :
  - `cd ../..`
  - `npm run dev`
  -  navagate to `http//localhost:5000`
 
  ![app](https://user-images.githubusercontent.com/92983658/178547477-e7729709-94ee-4f69-97dd-16280453e472.png)

