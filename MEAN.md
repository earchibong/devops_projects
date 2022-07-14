## MEAN STACK INSTALLATION ON UBUNTU

## Step One: Install Node JS & Node Package Manager (npm) with Apt Using NodeSource PPA

The NodeSource nodejs package contains both the node binary and npm, so no need to install npm separately.
- install PPA in home directory:
  - retrieve installation script for preferred node version: `curl -sL https://deb.nodesource.com/setup_16.x -o /tmp/nodesource_setup.sh`
  - inspect content of script: `nano /tmp/nodesource_setup.sh`
  - run script to add PPA: `sudo bash /tmp/nodesource_setup.sh`
  - add certificates: `sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`
  - install `Node.js` package: `sudo apt install nodejs`
 
## Step Two: Install MongoDB And Dependencies

- install certificates:
  - `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`
  - `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

![RECORDS](https://user-images.githubusercontent.com/92983658/178759715-fdf84168-68d3-4f8e-b64f-7b24fba2ceef.png)

- install MongoDB: `sudo apt-get install -y mongodb`
- start the server: `sudo service mongodb start`
- verify service: `sudo systemctl status mongodb`

![VERIFY](https://user-images.githubusercontent.com/92983658/178766934-72012e86-a284-4b8e-84c5-f501407ce06f.png)

- install body parser: `sudo npm install body-parser`
- install `Express` and `mongoose`: `sudo npm install express mongoose`


# Step Three: Build Server Side Application

- create a new directory `Books`: `mkdir Books`
- Initialize `npm` in `Books` and add a file named `server.js`:
  - `cd Books`
  - `npm init`
  - `touch server.js`
  
  ![INIT](https://user-images.githubusercontent.com/92983658/178772866-15b5eb80-1a31-421f-90dd-ddb98ef5d620.png)

- paste the following in `server.js`:
```

var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes');
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});

```

![SERVER](https://user-images.githubusercontent.com/92983658/178773398-246968fa-3607-4fc0-b6f2-89c88c60643e.png)


## Step Four: Create Server Routes

- in `Books` directory, create a new directory `apps` and a new file inside `apps` names `routes.js` : 
  - `cd Books`
  - `mkdir apps`
  - `cd apps & touch routes.js`

- in `routes.js` paste the following code:

```

var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};

```

![ROUTES](https://user-images.githubusercontent.com/92983658/178775740-c5a96ce9-0610-414e-b346-7d72d31845b9.png)

- in `apps` directory, create a new directory named `models` and creae a file named `books.js` inside:
  - `cd apps`
  - `mkdir models && cd models && touch book.js`

- paste the follwoing code inside `book.js`:

```

var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

```

![BOOKJS](https://user-images.githubusercontent.com/92983658/178777124-601287c4-8ac9-44db-83a8-4ec30efa0e4e.png)


## Step Five: Connect To Routes With Angular JS

- in Book directory, create a new directory named `public` and create file named `script.js`: 
  - `cd ../..`
  - `mkdir public`
  -  `touch script.js`
- paste the following in `script.js` file:

```

var app = angular.module('myApp', ["ngRoute"]);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

```

- create `index.html` file in `public` directory
- paste the following in `index.html`

```

<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

```

![INDEX](https://user-images.githubusercontent.com/92983658/178778865-01c4f4cd-3eae-44fd-930b-f0e474288efb.png)

 - go back to `Books` directory and start the server
  - `cd ..`
  - `node server.js`
  
  ![SERVER](https://user-images.githubusercontent.com/92983658/178781478-96e35189-0c7a-4ba0-8f6c-4a28ad35f26c.png)
  
  
- in EC2 security group, open `TCP port 3300`

![BROWSER](https://user-images.githubusercontent.com/92983658/178985099-3d5386ca-1495-4e7b-a25a-49a1986afe53.png)

