### Note: The MySQL database referred to in this repository is no longer operational. We have left the repo available for future reference.

# rdbms-mdb-migration-workshop

Additional help can be found at the site of a slightly different workshop https://github.com/mcinteerj/rdbms-mdb-migration-workshop

## The Challenge:

‘National Telecom’ is a leading Telecom Service Provider in the UK. They have decided to revamp their Customer Billing System as the current system is unable to perform well with the increasing amount of data related to customer calls. In addition, the company is currently unable to send targeted information about rate plans and services to customers based on their usage, due to which it is losing market share to competitors.  

You are in charge of: 
Migrating the system from the existing relational database to MongoDB
Supporting the marketing team by running analytical queries to study customer trends 
Presenting these results in a visual format for the business team’s perusal  

Features of the new Customer Billing System:
It is a web application, meant to be used internally by National Telecom employees only

The underlying database contains information about the company’s subscribers and their call history. 

Since active subscribers make calls on a regular basis, the amount of call history data linked to each subscriber is always growing

For this exercise, assume that each subscriber would have under 50 calls in their call history. However, in a real life scenario, the number of calls may be much more, which can result in an unbounded array.   


The majority of the queries on this system search for subscribers based on their Subscriber ID and retrieve fields related to their calling history like Call Timestamp, Call Duration and Current Rate Plan

The new system is being designed to also extend support to the marketing team. Some examples of additional queries to be supported are:

- Average call duration per month per age group of subscribers
- Most popular Rate Plans per city, etc. 

## Excercises

MongoDB Schema Design : Using the Relational Database Schema provided as reference for the dataset, design a MongoDB Schema for the new Customer Billing Application. 

Data Migration : Transfer the data from the existing Relational Database to MongoDB. 

Aggregation Pipeline : Using MongoDB’s Aggregation capabilities, perform the following analytics queries: 
* Find out the average call duration by Gender across all subscribers
* Find out the ‘Peak Calling Time’  by analysing the total number of calls made per hour by all subscribers
MongoDB Charts : Choose any of the Aggregation Queries from Exercise 2 and show it off in a visualisation of your choice using MongoDB Charts!


## Some help

Download the file node_modules-pg.tar.gz and then upload it to the 'dependencies' section in realm.  This is a pre-created dependancy archive https://docs.mongodb.com/realm/functions/upload-external-dependencies/

You can use the node-postgres module for connecting to our legacy database.

Connection can be acheived by using the connection:
```javascript
const { Client } = require('pg')

const client = new Client({
  user: 'readonly',
  host: 'rj-natwest-postgres.cbvyejbkhkfu.eu-west-1.rds.amazonaws.com',
  database: 'telecom',
  password: 'myreadonlypassword',
  port: 5432,
})

client.connect()
client.query('SELECT * from public."customers"', (err, res) => {
  console.log(err, JSON.stringify(res.rows));
  client.end();
}); 
```
More details - https://node-postgres.com/

The legacy database has the following structure:
```
CREATE TABLE IF NOT EXISTS public.customers
(
    subscriber_id integer NOT NULL,
    name character varying(80) COLLATE pg_catalog."default" NOT NULL,
    gender character varying(1) COLLATE pg_catalog."default" NOT NULL,
    email character varying(255) COLLATE pg_catalog."default" NOT NULL,
    phone_number character varying(40) COLLATE pg_catalog."default" NOT NULL,
    date_of_birth character varying(40) COLLATE pg_catalog."default" NOT NULL,
    street character varying(100) COLLATE pg_catalog."default" NOT NULL,
    zip character varying(10) COLLATE pg_catalog."default" NOT NULL,
    city character varying(100) COLLATE pg_catalog."default" NOT NULL,
    country_code character varying(15) COLLATE pg_catalog."default" NOT NULL,
    CONSTRAINT customers_pkey PRIMARY KEY (subscriber_id)
)

CREATE TABLE IF NOT EXISTS public.calls
(
    calls_id integer NOT NULL,
    subscriber_id integer NOT NULL,
    rate_plan_id integer NOT NULL,
    date_time_stamp character varying(30) COLLATE pg_catalog."default" NOT NULL,
    connected_party_number character varying(40) COLLATE pg_catalog."default" NOT NULL,
    call_duration integer NOT NULL,
    CONSTRAINT calls_pkey PRIMARY KEY (calls_id)
)

CREATE TABLE IF NOT EXISTS public.rate_plan
(
    rate_plan_id integer NOT NULL,
    description character varying(255) COLLATE pg_catalog."default" NOT NULL,
    type integer NOT NULL,
    CONSTRAINT rate_plan_pkey PRIMARY KEY (rate_plan_id)
)
```
Create a 'Linked Data Source' to your already created MongoDB cluster. https://docs.mongodb.com/realm/mongodb/link-a-data-source/

You can then work with your MongoDB collection using the following syntax
```javascript
var collection = context.services.get("mongodb-atlas").db("pgmigrate").collection("USERS");
collection.deleteMany({});
```

# Don't look at the below if you want to do the challenge yourself!
## If you want some extra help

The below is a working (although not very efficiant, see how it can be improved!) method to migrate the data from postgres to mongodb

```javascript
exports = function(arg){
  /*
    Accessing application's values:
    var x = context.values.get("value_name");

    Accessing a mongodb service:
    var collection = context.services.get("mongodb-atlas").db("dbname").collection("coll_name");
    collection.findOne({ owner_id: context.user.id }).then((doc) => {
      // do something with doc
    });

    To call other named functions:
    var result = context.functions.execute("function_name", arg1, arg2);

    Try running in the console below.
  */
  
  var collection = context.services.get("mongodb-atlas").db("pgmigrate").collection("USERS");
  collection.deleteMany({});

  
  const { Client } = require('pg')
  
  const client = new Client({
    user: 'readonly',
    host: 'rj-natwest-postgres.cbvyejbkhkfu.eu-west-1.rds.amazonaws.com',
    database: 'telecom',
    password: 'myreadonlypassword',
    port: 5432,
  })
  client.connect()
  client.query('SELECT * from public."customers"', (err, res) => {
    // console.log(err, JSON.stringify(res.rows))
    var result = JSON.stringify(res.rows);
  
    var customers = [];
    res.rows.forEach((element, index, array) => {
      var customer = {};
      customer._id = element.subscriber_id;
      customer.gender = element.gender;
      customer.name = element.name;
      customer.email = element.email;
      customer.phone_number = element.phone_number;
      customer.date_of_birth = new Date(element.date_of_birth);
      customer.address = {
        street:element.street,
        city:element.city,
        zip:element.zip,
        country_code:element.country_code
      }
      
      client.query('SELECT * from public."calls" where subscriber_id = ' + customer._id, (err, res) => {
        // console.log(err, JSON.stringify(res.rows))
        // var result = JSON.stringify(res.rows);
        customer.calls = res.rows;
        collection.insertOne(customer);
        if (index == array.length - 1) {
          client.end();
          console.log("Done");
        }
      });

    })
  
  })
  
  return "done";
};
```
