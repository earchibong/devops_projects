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
  - create a `React app - client ` inside `Todo`: `create-react-app client` 
  
  ![create_react_client](https://user-images.githubusercontent.com/92983658/178269789-870ba7b1-d769-46a3-a082-727296084d38.png)
  
  - creaate a folder for backend called `server` : `mkdir server` 
  - initialise `package.json` inside `server` : 
       `cd server` 
       `npm init` 
    
    ![initialise_json_in_server](https://user-images.githubusercontent.com/92983658/178271884-a8d5a043-b0b1-45d2-a01b-db44cf60c275.png)
    
  <li> install dependencies: `npm install mongodb express cors dotenv` </li>
  ![install_dependencies_in_server](https://user-images.githubusercontent.com/92983658/178272447-7c4edcff-7ea5-4bfc-a9cd-9ef9cb3d3f12.png)

  - create `server.js` in `server` folder : `touch server.js` 
  - paste the following code inside the `server.js` file:
    
      - `nano server.js` 
      - paste the following:
        `const express = require("express");
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
        });`
    
    ![server_js_file](https://user-images.githubusercontent.com/92983658/178273375-6cba67f6-69a0-465b-8b12-495b9b453b6f.png)
    
  
## Step Three: Connect To Database
## Step Four: Server API Endpoints
## Step Five: Setting Up React App
## Step Six: Create Components
## Step Seven: Connect Front End To Back End.
