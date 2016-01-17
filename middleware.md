# Middleware

## Find where a method call came from

Sometimes you want to allow certain service calls internally (like creating a new user) but not through the REST or websocket API. This can be done by adding the information in a middleware to the `request.feathers` object which will be merged into service call parameters:

```js
app.use(function(req, res, next) {
  req.feathers.external = 'rest';
  next();
});

app.configure(feathers.socketio(function(io) {
  io.use(function(socket, next) {
    // For websockets the feathers object does not exist by default
    if(!socket.feathers) {
      socket.feathers = {};
    }

    socket.feathers.external = 'socketio';
    next();
  });
}));


app.use('/todos', {
  get: function(id, params, callback) {
    if(!params.external) {
      return callback(null, {
        id: id,
        text: 'Do ' + id + '!'
      });
    }
    callback(new Error('External access not allowed'));
  }
});

var todoService = app.service('todos');
// Call .get without the external parameter set to get the result
todoService.get('laundry', {}, function(error, todo) {
  todo.text // -> 'Do laundry!'
});
```