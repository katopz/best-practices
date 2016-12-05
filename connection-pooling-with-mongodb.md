
[Source](https://www.compose.com/articles/connection-pooling-with-mongodb/ "Permalink to Connection Pooling with MongoDB")

# Connection Pooling with MongoDB

[ Share on Twitter ][1] [ Share on Facebook ][2] [ Share on Google+ ][3] [ Vote on Hacker News ][4] Published Nov 28, 2016

**_TL;DR_** _Making proper use of connection pooling can massively improve the performance of your MongoDB deployment._

At Compose we often get support tickets from customers asking about the performance of their MongoDB deployments. In many cases, these issues can be addressed with a straightforward adjustment to how your applications connect to your database.

## Connecting to MongoDB the easy, but inefficient way

Let's take a look at some basic Node.js code for an app connection to a MongoDB deployment. You can find your connection string in the Connection Info panel in your Compose deployment overview. It will look something like this:

![Connection String][5]

Drop a username and password in where indicated and you're ready to make connections.
    
```js
    var express = require('express');  
    var app = express();  
    var MongoClient = require('mongodb').MongoClient;  
    var assert = require('assert');
    
    // Connection URL
    var url = '[connectionString]]';
    
    // start server on port 3000
    app.listen(3000, '0.0.0.0', function() {  
      // print a message when the server starts listening
      console.log("server starting");
    });
    
    // Use connect method to connect to the server when the page is requested
    app.get('/', function(request, response) {  
      MongoClient.connect(url, function(err, db) {
        assert.equal(null, err);
        db.listCollections({}).toArray(function(err, collections) {
            assert.equal(null, err);
            collections.forEach(function(collection) {
                console.log(collection);
            });
            db.close();
        })
        response.send('Connected - see console for a list of available collections');
      });
    });
```

Here we define a simple app using `express`, and connect to a MongoDB database in our deployment using the provided connection string. We start the server running, then respond to page requests at the home page by connecting to the database and outputting a list of available collections. Then we close the connection and wait for the next request. Simple and straightforward.

The problem with this approach is that every time the home page is requested, we make a new connection to the database, and connections can be expensive things to create, especially where authentication is involved. A good way to cut down on your connections expenses is to use connection pooling.

## What is connection pooling

Think of your connections like a delivery service: if you are taking delivery of a hundred packages, it's going to take a long time and result in a lot of wasted effort if you have to sign for each parcel individually, take it upstairs, and wait for the courier to ring the doorbell for the next parcel, which you'll have to sign for, take upstairs etc.

It's better for you (and better for the courier) if you can answer the door once, and sign for as many parcels as you feel like carrying upstairs before returning for some more. That, in a nutshell, is what connection pooling offers.

All these individual connections can also cause problems if they are left idle instead of being closed after they have been used, as every open connection consumes some server RAM. In terms of your friendly courier, that would be like you having multiple doors and multiple couriers, and you not bothering to return to a door after taking a delivery. A courier isn't allowed to leave until you've closed the door and said goodbye to confirm the delivery has been accepted, so he just waits there. Meanwhile, other couriers keep arriving, until eventually they run out of doors and start queuing up behind the couriers who are already there. Eventually, some of them are going to get bored, and your parcels will be returned undelivered.

With connection pooling, rather than isolating connection requests, we can group them together. This way we can make fewer connection requests: once you're in the pool you're trusted as long as you stay in there, and will only need to re-authenticate and reconnect if you leave and then try to come back later.

## Sounds great - how do I use it?

We're going to make two changes to the way we connect to our database. Instead of having our app wait around for a request before connecting to the database we're going to have it connect when the application starts, and we're going to give ourselves a pool of connections to draw from as and when we need them.

Here we're using the [node-mongodb-native][6] driver, which like most available [MongoDB drivers][7] has an option that you can use to set the size of your connection pool. For this driver, it's called `poolSize`, and has a default value of 5. We can make use of the `poolsize` option by creating a database connection variable in advance, and letting the driver allocate available spaces as new connection requests come in:
    
```js
    // This is a global variable we'll use for handing the MongoDB client
    var mongodb;
    
    // Connection URL
    var url = '[connectionString]';
    
    // Create the db connection
    MongoClient.connect(url, function(err, db) {  
        assert.equal(null, err);
        mongodb=db;
        }
    );
```

To change the size of the connection pool from the default, we can pass `poolSize` in as an option:
    
```js
    // Create the database connection
    MongoClient.connect(url, {  
      poolSize: 10
      // other options can go here
    },function(err, db) {
        assert.equal(null, err);
        mongodb=db;
        }
    );
```

Now we have a connection ready and waiting. To use our new connection, we just need to make use of our new global variable, `mongodb` when a request is made:
    
```js
    // Use the connect method to connect to the server when the page is requested
    app.get('/', function(request, response) {  
        mongodb.listCollections({}).toArray(function(err, collections) {
            assert.equal(null, err);
            collections.forEach(function(collection) {
                console.log(collection);
            });
        })
        response.send('See console for a list of available collections');
    });
```

## Anything else?

No, not really. Just letting the driver handle the connection pool for you will give you a big boost over running with individual connection requests.

* * *

If you have any feedback about this or any other Compose article, drop the Compose Articles team a line at [articles@compose.com][8]. We're happy to hear from you.

[1]: https://twitter.com/share?text=Connection%20Pooling%20with%20MongoDB&amp;url=https://www.compose.com/articles/connection-pooling-with-mongodb/&amp;via=composeio
[2]: https://www.facebook.com/sharer/sharer.php?u=https://www.compose.com/articles/connection-pooling-with-mongodb/
[3]: https://plus.google.com/share?url=https://www.compose.com/articles/connection-pooling-with-mongodb/
[4]: http://news.ycombinator.com/submitlink?u=https://www.compose.com/articles/connection-pooling-with-mongodb/&amp;t=Connection Pooling with MongoDB
[5]: http://res.cloudinary.com/dyyck73ly/image/upload/v1479228710/hox6pputiucd8em9vdiz.png
[6]: https://github.com/mongodb/node-mongodb-native
[7]: https://docs.mongodb.com/ecosystem/drivers/
[8]: mailto:articles%40compose.com

  
