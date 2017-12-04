# apidemo

Following the steps to create the simple Orders demo

# Create API Blueprint in Apiary

1. go to apiary.io and logon using your credentials (you can create a new account for free if you don't have one)

2. Create a new API named Sample Orders (upper menu "Create New API")

3. Add the following markdown text:

FORMAT: 1A
HOST: http://oc-144-21-66-168.compute.oraclecloud.com:3000/ordersdemo

# Orders

Simple get orders collection

## Orders Collection [/orders]

### Get Orders [GET]

+ Response 200 (application/json)

        [
            {
                "orderId": "order 1",
                "total_price": 50,
                "line_items": [
                    {
                        "lineItem": 1,
                        "productName": "Billy Bookshelf",
                        "quantity": 1,
                        "price": 50,
                        "currency": "EUR"
                    }
                ]
            }
        ]

4. Test the API mock by clicking on the "Get Orders" resource on the right-hand side on the screen, then click on Try > Call resource. Also take note of the URL used (which is the Mock URL, i.e. https://private-b3bf03-orders78.apiary-mock.com/orders)

5. Add API blueprint to GitHub repository by clicking on "Settings" menu item under "Link your GitHub repository"

6. Logon to Oracle API Platform CS (i.e. https://host/apiplatform) and create a new API as following:

    a. Click on "Create API"
    b. Add values as following: "Name": Orders Demo , "version": 1.0, "Description": Sample orders API
    c. Edit "API Request", add "v1/ordersdemo" as the "API Endpoint URL"
    e. Edit "Service Request", click "Enter a URL" then enter the Apiary Orders Demo mock URL
    f. Click on the "Deployments" icon (second top-down on the left), then click "Deploy API", select a gateway and then Deploy
    g. Try the URL from your browser or an API tester

# Create Sample Orders Microservice

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

# Unit Test the API endpoint against the API definition using Dredd (open source tool by Apiary)

1. Go back to Apiary, open the Orders Demo and click on Tests > Tutorial

2. Execute the npm commands as exactly as indicated

Make sure you have Node and NPM installed.
$ npm install -g dredd

Initialize Dredd. Mind the privacy of API key.
$ dredd init -r apiary -j apiaryApiKey:5259748f13b1f60890ed5666c135b0d7 -j apiaryApiName:orders78

Run the test, reports appear here.
$ dredd

3. If no errors then all good to create container and push to Docker hub and then Deploy

# Package and Deploy

1. Create a docker file inside the node application directory

touch Dockerfile

2. Edit the file as following:

FROM node:argon

## Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

## Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

## Bundle app source
COPY . /usr/src/app

EXPOSE 3001
CMD [ "npm", "start" ]

3. Build the container

docker build -t <docker hub user>/<container namme> .

i.e.

docker build -t luisw19/ordersdemo .

4. Push the docker image to docker hub

docker login --username=<username>

docker push <image name>

5. Run the container from any docker-enabled machine with the command:

docker run <docker container image name>

docker run -d -p 3001:3001 luisw19/ordersdemo

and to stop, get docker container id by running "docker ps" and then

docker stop <container ID>

# Update the Service Endpoint in the Orders API Demo in the API Platform Management Console

1. Register the service in the platform by running the command from the same server that's running the Node App

curl -i -k -H "Content-Type: application/json" -H "Authorization: Basic d2VibG9naWM6SUszQVcxbjU=" -X POST https://oc-144-21-66-168.compute.oraclecloud.com:7202/apiplatform/management/v1/services -d '{ "name": "Orders Demo Service", "description": "Order service self-registration", "version": "1.0", "implementation": { "executions": { "request": [ "1" ], "response": [ "2" ] }, "policies": [ { "id": "1", "type": "o:BackendRequest", "version": "1.0", "config": { "endpoints": [ { "name" : "endpoint 1", "useProxy": false, "url": "http://oc-144-21-66-168.compute.oraclecloud.com:3001/ordersdemo" } ] } }, { "id": "2", "type": "o:BackendResponse", "version": "1.0", "config": {} } ] } }'

2. Open Orders API, Edit "Service Request", click "Select Service" then select "Orders Demo Service"

3. Re-deploy

4. Re-run Dredd
