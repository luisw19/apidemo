# apidemo

Following the steps to create the demo

1. Install NPM -follow instructions for your OS from https://nodejs.org/en/

2. Create package.json

npm init

Use as main file. server.js
Use following git repo: https://github.com/luisw19/node

3. Install Express as backend server

npm install --save express

4. Install body-parser to parse JSON payloads

npm install --save body-parser

5. Ensure all dependencies are installed in local “node_module” folder

npm install

6. Create server.js

touch server.js

6. Add the following code to test “Hello World”

var express     =   require("express");
var app         =   express();
var bodyParser  =   require("body-parser");
var router      =   express.Router();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({"extended" : false}));

//Root resource with hello world
router.get("/",function(req,res){
    res.json({"response" : "Hello Birmingham UKOUG"});
});

app.use('/',router);

app.listen(3000);
console.log("Listening to PORT 3000");

7. Start the server

npm start

8. Stop the server (just control C) and let’s now add a “GET Orders” resource. Add the following block after Hellow World root

//orders resource
router.get("/orders",function(req,res){
    res.json(
        [{
          "orderId": "order 1",
          "total_price": 50,
          "line_items": [{
            "lineItem": 1,
            "productName": "Billy Bookshelf",
            "quantity": 1,
            "price": 50,
            "currency": "EUR"
          }]
        }]
      );
});
