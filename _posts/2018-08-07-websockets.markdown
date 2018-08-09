---
layout: single
title: "A (very) brief introduction to WebSockets with Flask and Socket.IO"
date: 2018-08-07
categories: javascript websockets
---

WebSockets is a great technology with a lot of interesting applications. In short, you can say that WebSockets were created to provide low-latency, full-duplex connection between browser and server. Apart from the complicated technical slang, what that means is that the server can send the browser data without the browser explicitly asking for it, and that's what makes a lot of our modern web applications possible.

WebSockets are available in most modern browsers, so you shouldn't feel afraid to use them. All the networking theory that goes behind those can get a little bit complicated, but we won't get into that. Instead, this post will show the implementation of a simple application of WebSockets in the context of a real-time messaging app. As most things in life, I think you can understand this better by doing, so feel free to code along!

For this little project, we will be using **Flask** for our back-end (since it's a very lightweight web framework, and very easy to understand). So, we will also need the Flask-SocketIO library, so let's start by installing both of these:


```
pip install Flask
pip install flask-socketio
```

You can read the complete documentation for Flask-SocketIO by [clicking here.](https://flask-socketio.readthedocs.io/en/latest/)

Let's start by importing the necessary libraries and starting the Flask app:

```python
import os

from flask import Flask

from flask_socketio import SocketIO, emit

app = Flask(__name__)
app.config["SECRET_KEY"] = os.getenv("SECRET_KEY")
socketio = SocketIO(app)

```

Now we will start working with the sockets. The main use of WebSockets is to enable the back-end server to communicate in real-time with our client. They will do that by using sending and receiving messages, so we will preceed our Python functions with a decorator that comes with Flask-SocketIO, like this:

```python
@socketio.on("some message")

def some_function(parameter):
​	# define your function

```

The result of this is the following: when something on the client-side triggers the sending of some message (in our case, it's ```some message```), the back-end will be alerted and will execute "some_function". The client can send some data as well, and that will be passed as a parameter to the server.

Let's say we are building a messaging app. Maybe we want our server to know whenever a user sends some new message. We can implement something like that easily:

```python
@socketio.on("new message")

def handle_messages(data):
​	user = data["user"]
​	message = data["message"]
​	# do something with data
```

We have defined here that, whenever the server receives the event "new message", the function ```handle_messages``` should run, and we can extract the data that was sent to the server by using some keys (we have also defined that those keys will be called *user* and *message*).

That's it for the back-end. Now we need to work on the client-side.

We need to load **Socket.IO** (a Javascript library that implements WebSockets) for our app to run. You can add the following line between your HTML head tags:

```html

<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/socket.io/1.3.6/socket.io.min.js"></script>

```

We need that so we can use Socket.IO on our client-side. After that, we can use the following function:

```javascript
document.addEventListener('DOMContentLoaded', () => {
​	// Connect to websocket, you can just copy this line
​	var socket = io.connect(location.protocol + '//' + document.domain + ':' + location.port);
​	// When socket connects, do some stuff
​	socket.on('connect', () => {
​		// you can use javascript
        //here to get the user message from some HTML elements,
​		socket.emit('new message', {'user': user, 'message': message});
​	});
});
```

The main thing that you should understand from the code snippet above is that, after connecting to the websocket, you can use ```socket.emit()``` to send an event to the server, where the first argument of this function is the name of the event we want to send (the name is up to you to decide), and the second argument is a set of some key-value pairs that we can use to send useful data. For the purpose of this sample application, let's say we want to send the *username* of the user who's sending the message and the actual *message* that this user wants to send.

Inside our Python code, we can also send messages to the client. Like the following:

```python
def sampleFunction():
  data = {
    # define some key-value pairs in there
  }
  emit("event_name", data)
```

In the function above, we are sending an event called ```event_name``` to our client, together with some data. We can then add the following code inside our Javascript client-side application to handle this event and receive the data that the server is sending:

```javascript
socket.on('event_name', data => {
​	// use data to do something useful
});
```
See? It's a very neat thing to be able to communicate so easily between server and client, and in real-time! Meaning that you can update the content of the web page every time the server sends an event to the client.

And that's it. This post was supposed to be a very brief introduction to using WebSockets with the Javascript library **Socket.IO** and **Flask**. You can look at a very basic implementation I've made of a real-time messaging app using Socket.IO [in here](https://github.com/vinicius0197/teamtalk).
