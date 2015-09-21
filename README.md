# primus-route-handler
Declare your API via express router and communicate with it via Primus websocket.

### API
This module exports single function
```
var ws_api_handler = require('primus-route-handler');
```
which returns a callback for your api requests:
```
spark.on('api', ws_api_handler(spark, router));
```
You can use anything else instead of `'api'` here, but make sure
that client uses the same channel.

### Example
Server:
```js
// sample routes
var router = require('express').Router();
router.use(checkAuth);
router.get('/user/:id', function (req, res) {
	... get user info from database ...
	res.send(userinfo);
});

...
var ws_api_handler = require('primus-route-handler');

primus.on('connection', function (spark) {
		spark.on('api', ws_api_handler(spark, router));
});
```
Client:
```
var socket = new Primus('http://localhost:3000');
socket.send('api', {path: '/user/1234'}, function (err, userinfo) {
	if (err) {
		// handle error
		// err.status contains status code
		// err.statusText may contain further description
	}
	console.log('User info:', userinfo);
});
```

Alternatively you can specify path as a second argument:
```
socket.send('api', '/user/1234', cb);
```
Also you can specify method in the data object, if it matters for your routes:
Client:
```
socket.send('api', {path: '/user', method: 'post', user: {name: 'john', age: 30}}, cb);
```
Server:
```
router.post('/user', addUser)
```
Method defaults to `'get'`.

### Routes
You specify routes as usually with express 4 router
([route](https://www.npmjs.com/package/router) module should work too.)
In route callbacks `req` argument can be used to gather information about request
and `res` allows to send response and set status.

#### req
Available standard express fields are: `req.url`, `req.query`, `req.body`, `req.method`.
Additionally `req.spark` is available in case you want to do something specific and do not plan to use this route for ajax.

#### res
To send client response use `res.send(data)`. To set status use `res.status(code [, description]).send()`.
`res.send`, `res.end`, `res.json` are all the same function.

### License
MIT
