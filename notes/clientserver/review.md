<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. Review of Client &amp; Server Javascript</a>
<ul>
<li><a href="#sec-1-1">1.1. Core Ideas</a></li>
<li><a href="#sec-1-2">1.2. Our First Node Server</a></li>
<li><a href="#sec-1-3">1.3. More Complicated Node Servers</a>
<ul>
<li><a href="#sec-1-3-1">1.3.1. Handling Requests</a></li>
<li><a href="#sec-1-3-2">1.3.2. Handling URLs</a></li>
<li><a href="#sec-1-3-3">1.3.3. Microblogging: First try</a></li>
<li><a href="#sec-1-3-4">1.3.4. Microblogging: Try Two</a></li>
<li><a href="#sec-1-3-5">1.3.5. Making Data Persistent with Files</a></li>
<li><a href="#sec-1-3-6">1.3.6. Making Data Persistent with Orchestrate</a></li>
</ul>
</li>
<li><a href="#sec-1-4">1.4. Our First Express Server</a>
<ul>
<li><a href="#sec-1-4-1">1.4.1. An Aside: package.json</a></li>
<li><a href="#sec-1-4-2">1.4.2. Our First Express Server For Realzies</a></li>
</ul>
</li>
<li><a href="#sec-1-5">1.5. A Microblogging Express Server</a>
<ul>
<li><a href="#sec-1-5-1">1.5.1. Adding a Real Interface: html forms</a></li>
</ul>
</li>
<li><a href="#sec-1-6">1.6. Microblog Redux</a>
<ul>
<li><a href="#sec-1-6-1">1.6.1. JQuery and the Rise of the Dom</a></li>
<li><a href="#sec-1-6-2">1.6.2. One Last Thing: History API</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# Review of Client & Server Javascript<a id="sec-1" name="sec-1"></a>

So before we get into our final projects for this course, let's go ahead and review the basics of how to use Javascript for both server-side and client-side programming. We'll be going through a series of small examples, building up on themselves until we eventually have a small application with enough features to flex our new-found muscle. There will be suggested exercises throughout giving ideas for how to modify the examples or writing your own applications. 

## Core Ideas<a id="sec-1-1" name="sec-1-1"></a>

So first let us discuss what the major ideas of what we've covered since leaving the basics of the javascript language. Basically, we've gone from a world that consists only of javascript code existing in a vaccuum that can, occasionally, print things to the output we're now considering javascript as a part of an ecosystem for web development. 

The two main components for any application deployed over the network, web app or otherwise, are

-   the client code
-   the server code

where the client handles the display of data and controlling the user interface to the application and the server code handles the real *logic* of the application: the way data is stored and retrieved, what users can access what functionality, and keeping everything synced between the various clients.

In the case of the programs we're dealing with, both the client and server are going to be written in Javascript. Both sides don't have to be written in the same language, and I dare say that the most common case is that they are *not*. One of the reasons, I think, for Node's popularity is that you can take your skills in javascript for client side programming and use them for server side programming as well. It's an understandably nice feature. On the other hand, I think it might make the distinction between client and server unclear to newcomers! In this review I'll be attempting to elucidate where the different bits of code are running: the client's browser or a server process. A general rule of thumb, though, is that if the code is making the HTML interactive in some way then it must be running in the client browser and otherwise it is probably running on the server. Yes, even most templating like we've seen in Express is actually server-side code that *generates* HTML rather than scripts that are included in the browser along *with* the HTML.

We're not really making the Model-View-Controller distinction here, per se, but it shows up a little bit in terms of how we'll eventually refactor things a bit.

We'll more or less follow the progression

## Our First Node Server<a id="sec-1-2" name="sec-1-2"></a>

Our very first Node server is going to be rather simple: it'll be a kind of "echo" server that will spit back out to the client "You said: message-from-client". This is shockingly simple when it comes to Node and so let's just include the code below:

    var http = require('http');
    
    var server = http.createServer(function (req,res) {
        res.writeHead(200, {'Content-Type' : 'text/plain'});
        var msg = "";
        req.on('data', function (chunk) {
            msg = msg + chunk.toString();
        });
        req.on('end', function () {
            res.write("You said: " + msg);
            res.end();
        })
    });
    
    server.listen(3000);
    
    console.log("Listening on port 3000\n");

Now let's talk about what this actually means piece by piece. The first thing is that we need to include the HTTP library in line 1 when we say `var http = require('http')`. The next important line is that we actually create our server using the HTTP library function `http.createServer`. This might be a good time to review that when we make a module with `require` we're actually returning the value of `module.exports`, which is often but not always an object, so we're actually accessing the functions of the libraries as methods of the object created by the `require` statement. 

Here, we give `http.createServer` a single argument: a callback. This callback itself takes two arguments: the incoming request object and the pre-made response object that will be fed back to the client. Under the hood, node is going to build an object of the class `http.IncomingMessage` that holds the data for the request and an object of the class `http.ServerResponse` that will hold the response to be sent back to the server and then passes those to the callback. You don't particularly need to know what happens under the hood, I don't really know what the code looks like under the hood, but that's the point of a library: you interact with the interface not the implementation.

This wasn't the only way we could write this function, though. For example, since `http.createServer` returns an *object* then we could have collapsed a couple of those lines together with "method chaining" and instead said

    var http = require('http');
    
    http.createServer(function (req,res) {
        res.writeHead(200, {'Content-Type' : 'text/plain'});
        var msg = "";
        req.on('data', function (chunk) {
            msg = msg + chunk.toString();
        });
        req.on('end', function () {
            res.write("You said: " + msg);
            res.end();
        });
    }).listen(3000);
    
    console.log("Listening on port 3000\n");

We could also simplify things a bit by combining the `write` and `end` into one step and using method chaining on the `req` to assign multiple event handlers at once so then we get

    var http = require('http');
    
    http.createServer(function (req,res) {
        res.writeHead(200, {'Content-Type' : 'text/plain'});
        var msg = "";
        req.on('data', function (chunk) {
            msg = msg + chunk.toString();
        }).on('end', function () {
            res.end("You said: " + msg);
        });
    }).listen(3000, function () {
        console.log("Listening on port 3000\n");
    });

Beyond that, I'm not sure if there's really good ways to make it simpler without potentially just making it harder to read. 

## More Complicated Node Servers<a id="sec-1-3" name="sec-1-3"></a>

Now let's try building our way up into a more complete server that can handle a small "microblogging" style service, except we'll only be dealing with a single user just to simplify everything at first. 

### Handling Requests<a id="sec-1-3-1" name="sec-1-3-1"></a>

Our first lesson in making a more complicated server is how to deal with proper HTTP requests from the client. To review briefly, there are four major request methods that you'll need to deal with
-   GET, which is the basic request your browser makes whenever it loads a webpage. This is the request method that represents /get/ting data from the server for display. A simple GET shouldn't modify anything
-   POST, which is the main method for creating new data and often the method used by forms
-   PUT, which is similar to POST but semantically it's for creating *or* updating data as opposed to creation only
-   DELETE, which unsurprisingly signals that data should be deleted

in order to handle these request methods in just plain node, we simply need to dispatch over the method of the request. Let's try a simple server to demonstrate this

    var http = require('http');
    
    http.createServer(function (req,res){
        var method = req.method;
        if (method === "POST") {
            res.end("It was a POST");
        }
        else if(method === "PUT"){
            res.end("Puttin'");
        }
        else if(method === "GET") {
            res.end("Go Getter");
        }
        else if(method === "DELETE") {
            res.end("The end of all things");
        }
        else {
            res.end("Something other than the four we discussed")
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n");
    });

This is a very simple and perhaps silly example, but this is the basic structure of how we respond to different types of requests.

### Handling URLs<a id="sec-1-3-2" name="sec-1-3-2"></a>

The other skill we need to brush up on is how to dispatch over the url of the site, which we can do with using the url library in node in order to parse the url into pieces. The first thing we'll do is just make sure that we handle displaying the posts if we make a get request to the root.

    var http = require('http');
    var url = require('url');
    
    http.createServer(function (req,res) {
        var urlObj = url.parse(req.url,true);
        var urlPaths = urlObj.path.slice(1).split('/');
        if (urlPaths[0] === "thing") {
            res.end("That was a thing");
        }
        else if(urlPaths[0] === "stuff") {
            res.end("Here's some stuff")
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n")
    });

The actual structure of the object that `url.parse` returns is given here: <http://nodejs.org/docs/latest/api/url.html> The main thing we need to pay attention to here is that the `.path` property will give us, as a string, the rest of the url after the actual domain so for example "<http://mythingy.io/stuff/thing>" then `.path` will give us the string "/stuff/thing" and we can thus pop off the first "/" with a `.slice(1)` and then break it into an array with `.split('/')`. 

### Microblogging: First try<a id="sec-1-3-3" name="sec-1-3-3"></a>

So let's go ahead and try to take a first stab at our microblogging site. We'll be doing some very, very simple HTML generation that will look awful but hopefully be at least renderable. 

To start with we'll just try to display the result of our GET at the root

    var http = require('http');
    var url = require('url');
    
    var posts = ["stuff","more stuff", "many tiny posts"];
    
    http.createServer(function (req,res){
        var method = req.method;
        var urlObj = url.parse(req.url,true);
        var urlPath = urlObj.path.slice(1).split('/')[0];
        if (method === "GET" && urlPath==="") {
            res.writeHead(200,{"Content-Type" : "text/html"})
            res.write("<ul>");
            for(var p = 0; p < posts.length; p++){
                res.write("<li>" + posts[p] + "</li>");
            }
            res.write("</ul>");
            res.end();
        }
        else {
            res.end("Not a supported request");
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n");
    });

Go ahead and try running the server and seeing what happens if you navigate to localhost:3000. It should display the simple little html that we've written.

### Microblogging: Try Two<a id="sec-1-3-4" name="sec-1-3-4"></a>

Now, let's go ahead and try to write a version of the server that can handle taking in input as well. We'll also, in this case, be using hogan.js in order to do some templating and making this a little bit easier. Note that we'll be using straight-up hogan for templating and *not* using it as middle-ware because I want to demystify what's happening with these template engines a little bit.

So first before you actually try running anything you'll need to type this in your command line

    npm install hogan.js

Okay, so first we'll make a hogan template file for displaying posts that will also have a form that will let us add a post and then we'll handle that as well.

    <!DOCTYPE html>
    <html lang="en">
    <body>
      <h1>Make Post</h1>
      <form action="/addpost" method="post">
        <input name="post" placeholder="Say something" type="text" maxlength="140">
        <button type="submit">Post</button>
      </form>
      <h1>Posts</h1>
      <ul>
        {{#posts}}
        <li>{{.}}</li>
        {{/posts}}
      </ul>
    </body>
    </html>

and now we include it in our main code below

    var http = require('http');
    var url = require('url');
    var fs = require('fs');
    var hogan = require('hogan.js');
    
    var templateFile = fs.readFileSync('posts-1.html').toString();
    var template = hogan.compile(templateFile);
    
    var posts = [];
    
    function extractValue(str){
        // this function is for splitting the data returned by a form
        // we need to split it across the = sign
        var index = str.indexOf('=');
        return str.slice(index+1);
    }
    
    http.createServer(function (req,res) {
        var method = req.method;
        var urlObj = url.parse(req.url,true);
        var urlPath = urlObj.path.slice(1).split('/')[0];
    
        if(method === "GET" && urlPath===""){
            var html = template.render({posts : posts});
            res.writeHead(200,{"Content-Type" : "text/html"});
            res.end(html);
        }
        else if (method === "POST" && urlPath ==="addpost") {
            var tempPost = "";
            req.on("data", function (chunk) {
                tempPost = tempPost + chunk.toString();
            });
            req.on("end", function () {
                var html ='<a href="/">Go Back</a>'
                posts.push(extractValue(tempPost));
                res.writeHead(200,{"Content-Type" : "text/html"});
                res.end(html);
            });
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n")
    });

This code is a lot longer than our first attempt. Let's try to understand the logic of what's happening here. First off, we've got the dispatch over method and url path that we looked at earlier. So hopefully that's not too weird at this point. Now if we're just loading the main page, then we take the template file we loaded with the `fs.readFileSync` and compiled with `hogan.compile` and we then *render* it by passing in the object `{ posts : posts}` which turns it into a *string* that contains the HTML we'll send back to the client. So what's `{ posts : posts}` mean? Well, we're telling the template renderer that where we used the variables `posts` in the template it should have the values of the variables `posts` in our code. Now, these variables don't have to be named the same thing at all but I find it easier to remember if there's a consistency between the names of the variables-in-the-template and the variables-in-the-code. In our template we have a form that will pass along 

Let's also talk about what's happening with the *other* request that's being handled which is a POST request to the url path "addpost". Since we're using straight-up node without any extra libraries other than templating, that means our data retrieval is inherently asynchronous. As such in order to get data out of the request we need to write our data handlers for the `data` and `end` events. The `data` request just does what we've seen earlier: we push all of the data into some variable we've set aside for this purpose. When we're done retrieving data, i.e. when the `end` event happens that's when we actually do something. So what do we do? Well, we take the data returned by the form on the main page, split it into the actual *value* we want with our simple `extractValue` function. We need to do this because the form returns data in the form of "post=blahblahblah". Why? Well, it's because we had an input with the *name* attribute set to *post*. This is a good of time as any to point out that the form element of the template was set to have an *action* of "\\/addpost" because that's the URL path of the request we'll handle and the *method* set to POST.

Hopefully this helps explain what's happening in this code so far. To summarize we've covered so far:

-   How to make simple http servers
-   How to respond to different urls and http method types
-   How to use simple templates and compile them with Hogan.js
-   How to use forms to send data from the client to the server

### Making Data Persistent with Files<a id="sec-1-3-5" name="sec-1-3-5"></a>

You'll notice that every time you restart the server that the data you've entered disappears. That's because it isn't *persistent*. In order for the data to last beyond just the single program we need to add some way of storing the data permanently. We'll eventually use databases for this but, to start, we'll just go ahead and use good-ol' files. Since our posts are just a simple list we can do this datastructure very simply.

We'll continue using our template from before and *most* of the code will be the same. We're going to take the opportunity to review modules though and separate out our interface for the posts into a different file as follows

    var fs = require('fs');
    
    var filename = "posts.dat"
    
    function writeData (newPost) {
        fs.appendFileSync(filename,newPost+'\n');
    }
    
    function readData (){
        var str = fs.readFileSync(filename).toString();
        var temp = str.split('\n');
        temp.pop();
        return temp;
    }
    
    module.exports.writeData = writeData;
    module.exports.readData = readData;

We can now modify our previous code:

    var http = require('http');
    var url = require('url');
    var fs = require('fs');
    var hogan = require('hogan.js');
    var db = require('./filedb');
    var templateFile = fs.readFileSync('posts-1.html').toString();
    var template = hogan.compile(templateFile);
    
    var posts = [];
    
    function extractValue(str){
        // this function is for splitting the data returned by a form
        // we need to split it across the = sign
        var index = str.indexOf('=');
        return str.slice(index+1);
    }
    
    http.createServer(function (req,res) {
        var method = req.method;
        var urlObj = url.parse(req.url,true);
        var urlPath = urlObj.path.slice(1).split('/')[0];
    
        if(method === "GET" && urlPath===""){
            posts = db.readData();
            var html = template.render({posts : posts});
            res.writeHead(200,{"Content-Type" : "text/html"});
            res.end(html);
        }
        else if (method === "POST" && urlPath ==="addpost") {
            var tempPost = "";
            req.on("data", function (chunk) {
                tempPost = tempPost + chunk.toString();
            });
            req.on("end", function () {
                var html ='<a href="/">Go Back</a>'
                db.writeData(extractValue(tempPost));
                res.writeHead(200,{"Content-Type" : "text/html"});
                res.end(html);
            });
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n")
    });

### Making Data Persistent with Orchestrate<a id="sec-1-3-6" name="sec-1-3-6"></a>

Files are useful for persistence in a pinch, but there's a number of disadvantages. First off, if the format of your data changes at all then you'll need to rewrite your custom code for storing data in a file and retrieving it. Second, if we want to actually be able to usefully *search* through our data, which our current naive use of files cannot do, then we'll have to add a good bit of code in order to handle this. General databases, on the other hand, can store data in many different kinds of formats equally well and come with pre-built notions of search. This is a Good Thing in general. 

So we'll review our use of the Orchestrate API and corresponding Node library and show how to modify our code to work with that notion of persistence instead.

So first go ahead and run

    npm install orchestrate

Then make a new Orchestrate application through the dashboard. Mine is going to be named "clarissa-tutorial", and it's reasonable to follow the convention when naming yours, although you can name it anything you want up-to-and-including "a-cat-named-princess-ozma-fuzzy-butt" because let's be honest with ourselves that is an amazing name for a cat though a bit wordy for an application name.

We'll still keep the same template file, but we need to add a new file called `config.js` in which we'll keep our api key for our Orchestrate database.

    module.exports = [YOUR KEY HERE]

    var http = require('http');
    var url = require('url');
    var fs = require('fs');
    var hogan = require('hogan.js');
    // loading our API key from our config file
    var apiKey = require('./config');
    
    // loading our connection to 
    var db = require('orchestrate')(apiKey);
    
    var templateFile = fs.readFileSync('posts-1.html').toString();
    var template = hogan.compile(templateFile);
    
    var posts = [];
    
    function extractValue(str){
        // this function is for splitting the data returned by a form
        // we need to split it across the = sign
        var index = str.indexOf('=');
        return str.slice(index+1);
    }
    
    http.createServer(function (req,res) {
        var method = req.method;
        var urlObj = url.parse(req.url,true);
        var urlPath = urlObj.path.slice(1).split('/')[0];
    
        if(method === "GET" && urlPath===""){
            db.list('posts').then(function (results) {
                var prePosts = results.body.results;
                posts = prePosts.map(function (p) {
                    return p.value.text;
                });
                var html = template.render({posts : posts});
                res.writeHead(200,{"Content-Type" : "text/html"});
                res.end(html);
            }).fail(function (err) {
                console.log(err);
                res.end(err);
            });
        }
        else if (method === "POST" && urlPath ==="addpost") {
            var tempPost = "";
            req.on("data", function (chunk) {
                tempPost = tempPost + chunk.toString();
            });
            req.on("end", function () {
                var html ='<a href="/">Go Back</a>';
                var newKey = posts.length;
                db.put('posts',newKey.toString(), 
                       { "text" : extractValue(tempPost)}).then( function (r) {
                           res.writeHead(200,{"Content-Type" : "text/html"});
                           res.end(html);
                       });
            });
        }
    }).listen(3000, function () {
        console.log("Listening on port 3000\n")
    });

Before we move on to talking about using Express to make simpler servers let's talk about how we've had to change our code to work with Orchestrate. First off, we need to require Orchestrate with `require('orchestrate')` but then, strangely, we do this thing where we *apply* it to our API key immediately? I know this has confused some people so let's just explain with the following code sample

    function myPlus(a){
        return function (b) {
            return a+b;
        };
    }
    
    console.log(myPlus(1)(1)); //should equal 2

In other words, if we have functions that return other functions then we can pass the arguments one at a time in order to compute the final result. In our example, we can write a version of `+` that takes its two arguments one at a time. In formal terms, this is called "currying" named after Haskell Curry, but that's a digression for another time. So when we have the line `require('orchestrate')(apiKey)` it means that the `module.exports` that orchestrate returns is a *function* that eats the API key and gives us the connection to the database.

Rather than using our `readData` and `writeData` functions that we wrote ourselves for our file based persistence we instead use the built in Orchestrate functions `.list` and `.put`. These functions have fairly simple signatures. `.list` takes the name of a collection and then, well actually that's the key word now isn't it? It's not terribly useful at all unless we use the promise `.then` in order to set an action to take once the data within the collection has been retrieved. In this case, `.then` takes a callback which takes the result of the retrieval as its only argument. Orchestrate data is a little more complicated than what our previous data looked like. So when we retrieve that data we need to acces the `.results` property to get back an *array* of Orchestrate data objects, and then we need to extract the *text* property out of the *value* object and to do this for each element of the array we use the `.map` method that every array comes with.

Similarly when we write the data to the database we need to provide the collection, the key, and the data in a json format object. In this case we just include a single field, `text`, to the json object. 

## Our First Express Server<a id="sec-1-4" name="sec-1-4"></a>

So we've done an awful lot now with just basic Node and it's time to move on to doing things The Easier Way by using Express. Let's start with the very basics of an Express server, but we first need a bit of a digression.

### An Aside: package.json<a id="sec-1-4-1" name="sec-1-4-1"></a>

As we add more complicated functionality to our servers we'll need to add libraries, this means that we'll have our `package.json` file that we need to run in our directory before we actually try running our files. The package.json file we'll be using has the following contents

    {
        "name" : "tutorial",
        "description" : "our fair tutorial",
        "dependencies" : {
            "express" : "*",
            "consolidate" : "*",
            "morgan" : "*",
            "orchestrate" : "*",
            "q" : "*",
            "body-parser" : "*",
            "hogan.js" : "*"
        }
    }

Now we're using a little bit of bad form here because really we should specify version numbers and not just say "\*", which means use the latest version of the dependency, and in general you should pick particular versions or limit the versions somehow. 

Now that we have our package file go ahead and run the following command

    npm install

### Our First Express Server For Realzies<a id="sec-1-4-2" name="sec-1-4-2"></a>

We'll make a super simple echo server like our first basic Node server, but this time with Express to explain the basics of how you *start* an Express server.

    var express = require('express');
    var bodyparser = require('body-parser');
    
    var app = express();
    app.use(bodyparser.text());
    
    app.post('/', function (req, res) {
        res.send(200, req.body);
    }).listen(3000, function () {
        console.log("Listening on port 3000\n");
    });

## A Microblogging Express Server<a id="sec-1-5" name="sec-1-5"></a>

### Adding a Real Interface: html forms<a id="sec-1-5-1" name="sec-1-5-1"></a>

## Microblog Redux<a id="sec-1-6" name="sec-1-6"></a>

### JQuery and the Rise of the Dom<a id="sec-1-6-1" name="sec-1-6-1"></a>

So you'll notice that we're just sending back simple and completely unformatted HTML. We're going to start talking about client-side scripting, styling, and jQuery now. 

1.  Finding Elements in JQuery

2.  Changing Classes

3.  Event Handling

    Now we're going to move away from using html forms the old fashion way with http actions and, instead, rely entirely on javascript code to make our buttons and all that work without forcing any page reloads. 

### One Last Thing: History API<a id="sec-1-6-2" name="sec-1-6-2"></a>