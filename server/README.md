server.js is gonna store all of our Node.js code
***

Path
===

This is how we used to serve the "public" directory:

```javascript
console.log(__dirname + '/../public'); // -> /Users/Bruno/Desktop/node-chat-app/server/../public
```

It's unnecessary to go into "server" just to get out of it and *then* go into the public folder. We'd like to just go from the project folder right into the "public". In order to do that, we're gonna use the [Path](https://nodejs.org/api/path.html#path_path_join_paths) module that comes with Node:

```javascript
const path = require('path');

const publicPath = path.join(__dirname, '../public');
```

`.join()` takes your partial path and it joins them together. We're passing `__dirname` as first argument, and as the second argument we're specifying the relative path.

```javascript
console.log(publicPath); // -> /Users/Bruno/Desktop/node-chat-app/public
```
***
Configure our Express static middleware. This is gonna serve up the "public" folder:

```javascript
app.use(express.static(publicPath));
```


package.json
===

"start" tells Heroku how to start the application. <br>
"engine" tells Heroku which version of Node to use:
```json
  "scripts": {
    "start": "node server/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
    "engines": {
    "node": "8.9.1"
  }
```

Socket.io
===

Socket.io makes it dead simple to set up a server that supports web sockets and to create a front end that communicates with the server. Socket.io has a back end and front end library.

We need to integrate Socket.io into our existing web server. Currently we use Express to make our web server. We create a new a Express app, we configure our middleware and we call `app.listen`. Behind the scenes, Express is using a built-in Node module called "http" to create the server. We're gonna need to use "http" ourselves, configure Express to work with "http" *then* we'll be able to add a Socket.io support:

```javascript
const http = require('http'); 
const socketIO = require('socket.io'); 

const app = express();
var server = http.createServer(app);
var io = socketIO(server);

server.listen(3000);
```

First we loaded the "http" module, then on the `server` variable we created a server using the "http" library. The `createSever` was used all along behind the scenes; when we call `app.listen()` on our Express app, it literally calls this exact same method. <br>
`createServer` takes a function. This function looks really similar to one of the Express callbacks, with the `req` and `res` jazz and all. "http" is used behind the scenes for Express. They're integrated so much so that you can provide the `app` variable as the argument!

Now we're using the "http" server instead of the Express server. So instead of calling `app.listen`, we're gonna call `server.listen`.

The `io` variable is used to configure the server to also use Socket.io by passing it (the `server` var) as argument to `socketIO`.

With the `io` variable we can do anything we want in terms of emitting or listening to events. This is how we're gonna communicate between the server and the client.

***

```javascript
io.on('connection', socket => {
  console.log('new user connected');

  socket.emit('newEmail', {
    from: 'mike@example.com',
    text: 'hey. what is going on.',
    createdAt: 123
  });

  socket.on('createEmail', (newEmail) => {
    console.log('createEmail', newEmail);
  });

  socket.on('disconnect', () => {
    console.log('User was disconnected');
  });
});
```

`io.on()` lets you register and event listener. We can listen for a specific event and do something when that event happens. The most popular event is `connection`. This let's you listen for a new connection, meaning that a client connected to the server, and it lets you do something when that connection comes in. In order to do something, you provide a callback function as the second argument and this callback function is gonna get called with a `socket`. This `socket` argument is similar to the socket argument we have access to over index.js. This represents the individual socket, as opposed to all of the users connected to the server. 

Web sockets are a persistent technology, meaning the client and server both keep the communication channel open for as long as both of them want to. If the server shuts down, the client doesn't really have a choice. And the same for the client-server relationship. If I close the browser tab, the server cannot force me to keep the connection open. When a connection drops, the client still gonna try to reconnect. When we restart the server using `nodemon` there's about 1/4 of a second of time where the server is down and the client notices that. It tries to reconnect and eventually it reconnects. On the client we can also do something when we successfully connect to the server. 